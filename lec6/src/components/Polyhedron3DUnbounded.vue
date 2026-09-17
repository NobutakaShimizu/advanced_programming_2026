<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
let renderer: THREE.WebGLRenderer | null = null
let controls: OrbitControls | null = null
let animationId: number | null = null

// 非有界な多面体の制約:
// x + y <= 1  ... (1)
// x >= 0      ... (2)
// y >= 0      ... (3)
// z >= 0      ... (4)
// (z方向に非有界)

// 有界な部分（z = 0からz = 2まで）の頂点を計算
function calculatePolyhedronVertices() {
  const zMax = 2 // 表示範囲の上限
  const vertices: number[][] = []
  
  // z = 0の面の頂点
  vertices.push([0, 0, 0])      // 原点
  vertices.push([1, 0, 0])      // x軸上の点
  vertices.push([0, 1, 0])       // y軸上の点
  
  // z = zMaxの面の頂点
  vertices.push([0, 0, zMax])   // z軸上の点
  vertices.push([1, 0, zMax])   // x軸上の点
  vertices.push([0, 1, zMax])   // y軸上の点
  
  return vertices
}

// 多面体の面を構築
function buildFaces(vertices: number[][]) {
  // 頂点のインデックスを取得
  const getVertexIdx = (coords: number[]) => {
    return vertices.findIndex(v => 
      Math.abs(v[0] - coords[0]) < 0.001 &&
      Math.abs(v[1] - coords[1]) < 0.001 &&
      Math.abs(v[2] - coords[2]) < 0.001
    )
  }
  
  const originIdx = getVertexIdx([0, 0, 0])
  const xAxisIdx = getVertexIdx([1, 0, 0])
  const yAxisIdx = getVertexIdx([0, 1, 0])
  const zAxisBottomIdx = getVertexIdx([0, 0, 0])
  const zAxisTopIdx = getVertexIdx([0, 0, 2])
  const xAxisTopIdx = getVertexIdx([1, 0, 2])
  const yAxisTopIdx = getVertexIdx([0, 1, 2])
  
  const faces: number[][] = []
  
  // z = 0の面（三角形）
  if (originIdx >= 0 && xAxisIdx >= 0 && yAxisIdx >= 0) {
    faces.push([originIdx, xAxisIdx, yAxisIdx])
  }
  
  // z = zMaxの面（三角形）
  if (zAxisTopIdx >= 0 && xAxisTopIdx >= 0 && yAxisTopIdx >= 0) {
    faces.push([zAxisTopIdx, xAxisTopIdx, yAxisTopIdx])
  }
  
  // 側面1: x = 0の面（四角形）
  if (originIdx >= 0 && yAxisIdx >= 0 && yAxisTopIdx >= 0 && zAxisTopIdx >= 0) {
    faces.push([originIdx, yAxisIdx, yAxisTopIdx, zAxisTopIdx])
  }
  
  // 側面2: y = 0の面（四角形）
  if (originIdx >= 0 && xAxisIdx >= 0 && xAxisTopIdx >= 0 && zAxisTopIdx >= 0) {
    faces.push([originIdx, xAxisIdx, xAxisTopIdx, zAxisTopIdx])
  }
  
  // 側面3: x + y = 1の面（四角形）
  if (xAxisIdx >= 0 && yAxisIdx >= 0 && xAxisTopIdx >= 0 && yAxisTopIdx >= 0) {
    faces.push([xAxisIdx, yAxisIdx, yAxisTopIdx, xAxisTopIdx])
  }
  
  return faces
}

onMounted(() => {
  if (!containerRef.value) return

  // シーン作成
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0xf5f5f5)

  // カメラ作成
  const width = containerRef.value.clientWidth
  const height = containerRef.value.clientHeight
  camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000)
  camera.position.set(3, 3, 3)
  camera.lookAt(0.5, 0.5, 1)

  // レンダラー作成
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(width, height)
  renderer.setPixelRatio(window.devicePixelRatio)
  containerRef.value.appendChild(renderer.domElement)

  // OrbitControls
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.enableZoom = true
  controls.enablePan = true
  controls.autoRotate = false
  controls.target.set(0.5, 0.5, 1)
  controls.update()

  // グリッド追加
  const gridSize = 3
  const gridDivisions = 30
  const gridHelper = new THREE.GridHelper(gridSize, gridDivisions, 0x888888, 0xcccccc)
  gridHelper.position.set(0, 0, 0)
  scene.add(gridHelper)

  // 軸追加
  const axesHelper = new THREE.AxesHelper(2)
  scene.add(axesHelper)

  // 多面体の頂点を計算
  const vertices = calculatePolyhedronVertices()
  
  // 頂点を可視化
  if (vertices.length > 0) {
    const pointsGeometry = new THREE.BufferGeometry()
    const positions = new Float32Array(vertices.flat())
    pointsGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
    const pointsMaterial = new THREE.PointsMaterial({ 
      color: 0xff0000, 
      size: 8,
      sizeAttenuation: false
    })
    const points = new THREE.Points(pointsGeometry, pointsMaterial)
    scene.add(points)
  }

  // 多面体の面を構築
  const faces = buildFaces(vertices)
  
  // 多面体を可視化
  if (faces.length > 0 && vertices.length > 0) {
    const geometry = new THREE.BufferGeometry()
    const positions: number[] = []
    const indices: number[] = []
    
    vertices.forEach((v) => {
      positions.push(...v)
    })
    
    // 三角形と四角形の面を処理
    faces.forEach(face => {
      if (face.length === 3) {
        // 三角形
        indices.push(face[0], face[1], face[2])
      } else if (face.length === 4) {
        // 四角形（2つの三角形に分割）
        indices.push(face[0], face[1], face[2])
        indices.push(face[0], face[2], face[3])
      }
    })
    
    geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
    geometry.setIndex(indices)
    geometry.computeVertexNormals()
    
    const material = new THREE.MeshBasicMaterial({
      color: 0xff9800,
      side: THREE.DoubleSide,
      opacity: 0.6,
      transparent: true,
      wireframe: true
    })
    
    const mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)
    
    // ソリッド版も追加
    const solidMaterial = new THREE.MeshPhongMaterial({
      color: 0xff9800,
      side: THREE.DoubleSide,
      opacity: 0.3,
      transparent: true,
      wireframe: false
    })
    const solidMesh = new THREE.Mesh(geometry, solidMaterial)
    scene.add(solidMesh)
  }

  // 平面を可視化
  const planeSize = 3
  
  // 平面1: x + y = 1 (法線ベクトル: (1, 1, 0))
  const plane1NormalRaw = new THREE.Vector3(1, 1, 0)
  const plane1Normal = plane1NormalRaw.clone().normalize()
  const plane1D = 1
  const plane1Distance = plane1D / plane1NormalRaw.length()
  const plane1Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane1Material = new THREE.MeshBasicMaterial({ 
    color: 0xff9800, 
    side: THREE.DoubleSide,
    opacity: 0.3,
    transparent: true,
    wireframe: false
  })
  const plane1 = new THREE.Mesh(plane1Geometry, plane1Material)
  const plane1Center = plane1Normal.clone().multiplyScalar(plane1Distance)
  plane1.position.copy(plane1Center)
  plane1.position.z = 1 // z方向の中央に配置
  const up = new THREE.Vector3(0, 0, 1)
  const quaternion1 = new THREE.Quaternion().setFromUnitVectors(up, plane1Normal)
  plane1.setRotationFromQuaternion(quaternion1)
  scene.add(plane1)

  // 平面2: x = 0
  const plane2Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane2Material = new THREE.MeshBasicMaterial({ 
    color: 0x4caf50, 
    side: THREE.DoubleSide,
    opacity: 0.3,
    transparent: true
  })
  const plane2 = new THREE.Mesh(plane2Geometry, plane2Material)
  plane2.position.set(0, 1.5, 1)
  plane2.rotation.y = Math.PI / 2
  scene.add(plane2)

  // 平面3: y = 0
  const plane3Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane3Material = new THREE.MeshBasicMaterial({ 
    color: 0x2196f3, 
    side: THREE.DoubleSide,
    opacity: 0.3,
    transparent: true
  })
  const plane3 = new THREE.Mesh(plane3Geometry, plane3Material)
  plane3.position.set(1.5, 0, 1)
  plane3.rotation.x = Math.PI / 2
  scene.add(plane3)

  // z方向に非有界であることを示す矢印（上方向）
  const arrowUp = new THREE.ArrowHelper(
    new THREE.Vector3(0, 0, 1),
    new THREE.Vector3(0.3, 0.3, 2.5),
    0.8,
    0xff9800,
    0.15,
    0.08
  )
  scene.add(arrowUp)

  // z方向に非有界であることを示すテキスト（簡易版：線で表現）
  // "∞" の代わりに、上方向の矢印を追加で表示
  const arrowUp2 = new THREE.ArrowHelper(
    new THREE.Vector3(0, 0, 1),
    new THREE.Vector3(0.7, 0.3, 2.5),
    0.6,
    0xff9800,
    0.12,
    0.06
  )
  scene.add(arrowUp2)

  // 座標軸の矢印
  const xAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(1, 0, 0),
    new THREE.Vector3(0, 0, 0),
    2,
    0xff0000,
    0.1,
    0.05
  )
  scene.add(xAxisLine)
  
  const yAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(0, 1, 0),
    new THREE.Vector3(0, 0, 0),
    2,
    0x00ff00,
    0.1,
    0.05
  )
  scene.add(yAxisLine)
  
  const zAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(0, 0, 1),
    new THREE.Vector3(0, 0, 0),
    2,
    0x0000ff,
    0.1,
    0.05
  )
  scene.add(zAxisLine)

  // ライト追加（ソリッド表示のため）
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambientLight)
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4)
  directionalLight.position.set(5, 5, 5)
  scene.add(directionalLight)

  // リサイズハンドラー
  const handleResize = () => {
    if (!containerRef.value || !camera || !renderer) return
    const width = containerRef.value.clientWidth
    const height = containerRef.value.clientHeight
    camera.aspect = width / height
    camera.updateProjectionMatrix()
    renderer.setSize(width, height)
  }
  window.addEventListener('resize', handleResize)

  // アニメーションループ
  const animate = () => {
    animationId = requestAnimationFrame(animate)
    if (controls) controls.update()
    if (renderer && scene && camera) {
      renderer.render(scene, camera)
    }
  }
  animate()

  // クリーンアップ
  onUnmounted(() => {
    window.removeEventListener('resize', handleResize)
  })
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  if (renderer && containerRef.value && renderer.domElement.parentNode) {
    containerRef.value.removeChild(renderer.domElement)
    renderer.dispose()
  }
})
</script>

<template>
  <div ref="containerRef" style="width: 100%; height: 400px; position: relative; border: 1px solid #ddd; border-radius: 8px;"></div>
</template>


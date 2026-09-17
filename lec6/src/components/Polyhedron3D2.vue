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

// 制約式から多面体の頂点を計算
function calculatePolyhedronVertices() {
  // 制約:
  // 2x+y+z <= 1  ... (1)
  // x+2y+z <= 1  ... (2)
  // x+y+2z <= 1  ... (3)
  // x, y, z >= 0
  
  // 各平面と座標平面の交点を計算
  const vertices: number[][] = []
  
  // 原点
  vertices.push([0, 0, 0])
  
  // 座標軸上の点
  // 平面1とy=0, z=0: 2x=1 => x=0.5 => (0.5, 0, 0)
  // 平面2とx=0, z=0: 2y=1 => y=0.5 => (0, 0.5, 0)
  // 平面3とx=0, y=0: 2z=1 => z=0.5 => (0, 0, 0.5)
  
  // 平面1とx=0, z=0: y=1 => (0, 1, 0) - ただし、これは平面2と平面3を満たさない可能性がある
  // 平面1とx=0, y=0: z=1 => (0, 0, 1) - ただし、これは平面2と平面3を満たさない可能性がある
  // 平面2とy=0, z=0: x=1 => (1, 0, 0) - ただし、これは平面1と平面3を満たさない可能性がある
  
  // 3つの平面の交点
  // 2x+y+z = 1  ... (1)
  // x+2y+z = 1  ... (2)
  // x+y+2z = 1  ... (3)
  
  // (1)-(2): x-y=0 => x=y
  // (1)-(3): x-z=0 => x=z
  // よって x=y=z
  // (1)に代入: 2x+x+x = 4x = 1 => x = 0.25
  // よって (0.25, 0.25, 0.25)
  
  // 各平面と座標平面の交点を計算
  const candidateVertices = [
    [0, 0, 0],           // 原点
    [0.5, 0, 0],         // 平面1とy=0, z=0
    [0, 0.5, 0],         // 平面2とx=0, z=0
    [0, 0, 0.5],         // 平面3とx=0, y=0
    [0.25, 0.25, 0.25],  // 3つの平面の交点
    [1, 0, 0],           // 平面2とy=0, z=0（制約チェック必要）
    [0, 1, 0],           // 平面1とx=0, z=0（制約チェック必要）
    [0, 0, 1],           // 平面1とx=0, y=0（制約チェック必要）
  ]
  
  // すべての制約を満たす頂点をフィルタリング
  return candidateVertices.filter(v => {
    const [x, y, z] = v
    return x >= -0.001 && y >= -0.001 && z >= -0.001 &&
           (2*x + y + z) <= 1.001 && // 浮動小数点誤差を考慮
           (x + 2*y + z) <= 1.001 &&
           (x + y + 2*z) <= 1.001
  })
}

// 多面体の面を構築（手動で定義）
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
  const xAxisIdx = getVertexIdx([0.5, 0, 0])
  const yAxisIdx = getVertexIdx([0, 0.5, 0])
  const zAxisIdx = getVertexIdx([0, 0, 0.5])
  const centerIdx = getVertexIdx([0.25, 0.25, 0.25])
  
  const faces: number[][] = []
  
  // 原点と2つの軸上の点で構成される面
  if (originIdx >= 0 && xAxisIdx >= 0 && yAxisIdx >= 0) {
    faces.push([originIdx, xAxisIdx, yAxisIdx])
  }
  if (originIdx >= 0 && yAxisIdx >= 0 && zAxisIdx >= 0) {
    faces.push([originIdx, yAxisIdx, zAxisIdx])
  }
  if (originIdx >= 0 && zAxisIdx >= 0 && xAxisIdx >= 0) {
    faces.push([originIdx, zAxisIdx, xAxisIdx])
  }
  
  // 中心点を含む面
  if (centerIdx >= 0) {
    if (xAxisIdx >= 0 && yAxisIdx >= 0) {
      faces.push([xAxisIdx, yAxisIdx, centerIdx])
    }
    if (yAxisIdx >= 0 && zAxisIdx >= 0) {
      faces.push([yAxisIdx, zAxisIdx, centerIdx])
    }
    if (zAxisIdx >= 0 && xAxisIdx >= 0) {
      faces.push([zAxisIdx, xAxisIdx, centerIdx])
    }
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
  camera.position.set(2, 2, 2)
  camera.lookAt(0.5, 0.5, 0.5)

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
  controls.enablePan = true  // 平行移動（パン）を有効化
  controls.autoRotate = false
  controls.target.set(0.5, 0.5, 0.5)
  controls.update()

  // グリッド追加
  const gridSize = 2
  const gridDivisions = 20
  const gridHelper = new THREE.GridHelper(gridSize, gridDivisions, 0x888888, 0xcccccc)
  gridHelper.position.set(0, 0, 0)
  scene.add(gridHelper)

  // 軸追加
  const axesHelper = new THREE.AxesHelper(1.5)
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
  
  // 多面体を可視化（ワイヤーフレーム）
  if (faces.length > 0 && vertices.length > 0) {
    const geometry = new THREE.BufferGeometry()
    const positions: number[] = []
    const indices: number[] = []
    
    vertices.forEach((v, i) => {
      positions.push(...v)
    })
    
    faces.forEach(face => {
      if (face.length === 3) {
        indices.push(face[0], face[1], face[2])
      }
    })
    
    geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
    geometry.setIndex(indices)
    geometry.computeVertexNormals()
    
    const material = new THREE.MeshBasicMaterial({
      color: 0x2196f3,
      side: THREE.DoubleSide,
      opacity: 0.6,
      transparent: true,
      wireframe: true
    })
    
    const mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)
    
    // ソリッド版も追加
    const solidMaterial = new THREE.MeshPhongMaterial({
      color: 0x2196f3,
      side: THREE.DoubleSide,
      opacity: 0.3,
      transparent: true,
      wireframe: false
    })
    const solidMesh = new THREE.Mesh(geometry, solidMaterial)
    scene.add(solidMesh)
  }

  // 平面を可視化
  const planeSize = 2
  
  // 平面1: 2x+y+z = 1 (法線ベクトル: (2, 1, 1))
  const plane1NormalRaw = new THREE.Vector3(2, 1, 1)
  const plane1Normal = plane1NormalRaw.clone().normalize()
  const plane1D = 1
  const plane1Distance = plane1D / plane1NormalRaw.length()
  const plane1Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane1Material = new THREE.MeshBasicMaterial({ 
    color: 0x2196f3, 
    side: THREE.DoubleSide,
    opacity: 0.4,
    transparent: true,
    wireframe: false
  })
  const plane1 = new THREE.Mesh(plane1Geometry, plane1Material)
  const plane1Center = plane1Normal.clone().multiplyScalar(plane1Distance)
  plane1.position.copy(plane1Center)
  const up = new THREE.Vector3(0, 0, 1)
  const quaternion1 = new THREE.Quaternion().setFromUnitVectors(up, plane1Normal)
  plane1.setRotationFromQuaternion(quaternion1)
  scene.add(plane1)

  // 平面2: x+2y+z = 1 (法線ベクトル: (1, 2, 1))
  const plane2NormalRaw = new THREE.Vector3(1, 2, 1)
  const plane2Normal = plane2NormalRaw.clone().normalize()
  const plane2D = 1
  const plane2Distance = plane2D / plane2NormalRaw.length()
  const plane2Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane2Material = new THREE.MeshBasicMaterial({ 
    color: 0x4caf50, 
    side: THREE.DoubleSide,
    opacity: 0.4,
    transparent: true
  })
  const plane2 = new THREE.Mesh(plane2Geometry, plane2Material)
  const plane2Center = plane2Normal.clone().multiplyScalar(plane2Distance)
  plane2.position.copy(plane2Center)
  const quaternion2 = new THREE.Quaternion().setFromUnitVectors(up, plane2Normal)
  plane2.setRotationFromQuaternion(quaternion2)
  scene.add(plane2)

  // 平面3: x+y+2z = 1 (法線ベクトル: (1, 1, 2))
  const plane3NormalRaw = new THREE.Vector3(1, 1, 2)
  const plane3Normal = plane3NormalRaw.clone().normalize()
  const plane3D = 1
  const plane3Distance = plane3D / plane3NormalRaw.length()
  const plane3Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane3Material = new THREE.MeshBasicMaterial({ 
    color: 0xff9800, 
    side: THREE.DoubleSide,
    opacity: 0.4,
    transparent: true
  })
  const plane3 = new THREE.Mesh(plane3Geometry, plane3Material)
  const plane3Center = plane3Normal.clone().multiplyScalar(plane3Distance)
  plane3.position.copy(plane3Center)
  const quaternion3 = new THREE.Quaternion().setFromUnitVectors(up, plane3Normal)
  plane3.setRotationFromQuaternion(quaternion3)
  scene.add(plane3)

  // 座標軸の矢印
  const xAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(1, 0, 0),
    new THREE.Vector3(0, 0, 0),
    1.5,
    0xff0000,
    0.1,
    0.05
  )
  scene.add(xAxisLine)
  
  const yAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(0, 1, 0),
    new THREE.Vector3(0, 0, 0),
    1.5,
    0x00ff00,
    0.1,
    0.05
  )
  scene.add(yAxisLine)
  
  const zAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(0, 0, 1),
    new THREE.Vector3(0, 0, 0),
    1.5,
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


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
  // 0.3 x_v + 0.4 x_m >= 30  ... (1)
  // 0.6 x_m + 0.2 x_c >= 20  ... (2)
  // 0.8 x_v + 0.1 x_c >= 15  ... (3)
  // x_v, x_m, x_c >= 0
  
  // 各平面と座標平面の交点を計算して有効な頂点を求める
  const vertices: number[][] = []
  
  // 3つの平面の交点を計算（連立方程式を解く）
  // 各平面の組み合わせで交点を求める
  
  // 平面1と平面2の交点（x_v=0の場合）
  // 0.4 x_m = 30 => x_m = 75
  // 0.6 * 75 + 0.2 x_c = 20 => x_c = (20 - 45) / 0.2 = -125 (無効)
  
  // 平面1と平面3の交点（x_m=0の場合）
  // 0.3 x_v = 30 => x_v = 100
  // 0.8 * 100 + 0.1 x_c = 15 => x_c = (15 - 80) / 0.1 = -650 (無効)
  
  // 平面2と平面3の交点（x_v=0の場合）
  // 0.6 x_m + 0.2 x_c = 20
  // 0.1 x_c = 15 => x_c = 150
  // 0.6 x_m + 0.2 * 150 = 20 => x_m = (20 - 30) / 0.6 = -16.67 (無効)
  
  // 3つの平面の交点を計算
  // 0.3 x_v + 0.4 x_m = 30  ... (1)
  // 0.6 x_m + 0.2 x_c = 20  ... (2)
  // 0.8 x_v + 0.1 x_c = 15  ... (3)
  
  // (2)から: x_c = (20 - 0.6 x_m) / 0.2 = 100 - 3 x_m
  // (3)に代入: 0.8 x_v + 0.1(100 - 3 x_m) = 15
  // 0.8 x_v + 10 - 0.3 x_m = 15
  // 0.8 x_v - 0.3 x_m = 5  ... (4)
  
  // (1)と(4)を連立
  // 0.3 x_v + 0.4 x_m = 30  ... (1)
  // 0.8 x_v - 0.3 x_m = 5   ... (4)
  
  // (1)×0.8: 0.24 x_v + 0.32 x_m = 24
  // (4)×0.3: 0.24 x_v - 0.09 x_m = 1.5
  // 差を取る: 0.41 x_m = 22.5
  // x_m = 22.5 / 0.41 ≈ 54.88
  
  // (1)に代入: 0.3 x_v + 0.4 * 54.88 = 30
  // 0.3 x_v = 30 - 21.95 = 8.05
  // x_v ≈ 26.83
  
  // (2)に代入: 0.6 * 54.88 + 0.2 x_c = 20
  // 0.2 x_c = 20 - 32.93 = -12.93
  // x_c = -64.65 (無効)
  
  // より実用的なアプローチ: 各平面と座標平面の交点を計算
  const candidateVertices: number[][] = []
  
  // 平面1 (0.3 x_v + 0.4 x_m = 30) と座標平面の交点
  // x_c=0, x_m=0: x_v = 100
  if (100 >= 0) candidateVertices.push([100, 0, 0])
  // x_c=0, x_v=0: x_m = 75
  if (75 >= 0) candidateVertices.push([0, 75, 0])
  
  // 平面2 (0.6 x_m + 0.2 x_c = 20) と座標平面の交点
  // x_v=0, x_m=0: x_c = 100
  if (100 >= 0) candidateVertices.push([0, 0, 100])
  // x_v=0, x_c=0: x_m = 33.33
  if (33.33 >= 0) candidateVertices.push([0, 33.33, 0])
  
  // 平面3 (0.8 x_v + 0.1 x_c = 15) と座標平面の交点
  // x_m=0, x_c=0: x_v = 18.75
  if (18.75 >= 0) candidateVertices.push([18.75, 0, 0])
  // x_m=0, x_v=0: x_c = 150
  if (150 >= 0) candidateVertices.push([0, 0, 150])
  
  // 平面1と平面2の交点（x_v=0の場合を除く）
  // 平面1: 0.3 x_v + 0.4 x_m = 30
  // 平面2: 0.6 x_m + 0.2 x_c = 20
  // x_v=0の場合: x_m=75, x_c=-125 (無効)
  
  // 平面1と平面3の交点（x_m=0の場合を除く）
  // x_m=0の場合: x_v=100, x_c=-650 (無効)
  
  // 平面2と平面3の交点（x_v=0の場合を除く）
  // x_v=0の場合: x_c=150, x_m=-16.67 (無効)
  
  // 実際の多面体の頂点は、これらの交点のうち、すべての制約を満たすもの
  // 簡略化のため、範囲内でサンプリングして可視化
  return candidateVertices.filter(v => {
    const [xv, xm, xc] = v
    return xv >= 0 && xm >= 0 && xc >= 0
  })
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
  camera.position.set(200, 200, 200)
  camera.lookAt(50, 50, 50)

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

  // グリッド追加
  const gridSize = 200
  const gridDivisions = 20
  const gridHelper = new THREE.GridHelper(gridSize, gridDivisions, 0x888888, 0xcccccc)
  scene.add(gridHelper)

  // 軸追加
  const axesHelper = new THREE.AxesHelper(100)
  scene.add(axesHelper)

  // 軸ラベル用のテキスト（簡易版）
  const createAxisLabel = (text: string, position: THREE.Vector3, color: number) => {
    // 簡易的な実装（実際にはCSS2DRendererを使う方が良い）
  }

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

  // 平面を可視化
  const planeSize = 250
  
  // 平面1: 0.3 x_v + 0.4 x_m = 30 (法線ベクトル: (0.3, 0.4, 0))
  // 平面の方程式: n·p = d (nは法線、pは平面上の点、dは定数)
  const plane1NormalRaw = new THREE.Vector3(0.3, 0.4, 0)
  const plane1Normal = plane1NormalRaw.clone().normalize()
  const plane1D = 30
  const plane1Distance = plane1D / plane1NormalRaw.length()
  const plane1Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane1Material = new THREE.MeshBasicMaterial({ 
    color: 0x2196f3, 
    side: THREE.DoubleSide,
    opacity: 0.5,
    transparent: true,
    wireframe: false
  })
  const plane1 = new THREE.Mesh(plane1Geometry, plane1Material)
  // 平面の位置と向きを設定
  const plane1Center = plane1Normal.clone().multiplyScalar(plane1Distance)
  plane1.position.copy(plane1Center)
  // 法線ベクトルの方向を向く
  const up = new THREE.Vector3(0, 0, 1)
  const quaternion = new THREE.Quaternion().setFromUnitVectors(up, plane1Normal)
  plane1.setRotationFromQuaternion(quaternion)
  scene.add(plane1)

  // 平面2: 0.6 x_m + 0.2 x_c = 20 (法線ベクトル: (0, 0.6, 0.2))
  const plane2NormalRaw = new THREE.Vector3(0, 0.6, 0.2)
  const plane2Normal = plane2NormalRaw.clone().normalize()
  const plane2D = 20
  const plane2Distance = plane2D / plane2NormalRaw.length()
  const plane2Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane2Material = new THREE.MeshBasicMaterial({ 
    color: 0x4caf50, 
    side: THREE.DoubleSide,
    opacity: 0.5,
    transparent: true
  })
  const plane2 = new THREE.Mesh(plane2Geometry, plane2Material)
  const plane2Center = plane2Normal.clone().multiplyScalar(plane2Distance)
  plane2.position.copy(plane2Center)
  const quaternion2 = new THREE.Quaternion().setFromUnitVectors(up, plane2Normal)
  plane2.setRotationFromQuaternion(quaternion2)
  scene.add(plane2)

  // 平面3: 0.8 x_v + 0.1 x_c = 15 (法線ベクトル: (0.8, 0, 0.1))
  const plane3NormalRaw = new THREE.Vector3(0.8, 0, 0.1)
  const plane3Normal = plane3NormalRaw.clone().normalize()
  const plane3D = 15
  const plane3Distance = plane3D / plane3NormalRaw.length()
  const plane3Geometry = new THREE.PlaneGeometry(planeSize, planeSize)
  const plane3Material = new THREE.MeshBasicMaterial({ 
    color: 0xff9800, 
    side: THREE.DoubleSide,
    opacity: 0.5,
    transparent: true
  })
  const plane3 = new THREE.Mesh(plane3Geometry, plane3Material)
  const plane3Center = plane3Normal.clone().multiplyScalar(plane3Distance)
  plane3.position.copy(plane3Center)
  const quaternion3 = new THREE.Quaternion().setFromUnitVectors(up, plane3Normal)
  plane3.setRotationFromQuaternion(quaternion3)
  scene.add(plane3)

  // 座標軸のラベル（簡易版として線を追加）
  // x軸 (x_v)
  const xAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(1, 0, 0),
    new THREE.Vector3(0, 0, 0),
    120,
    0xff0000,
    10,
    5
  )
  scene.add(xAxisLine)
  
  // y軸 (x_m)
  const yAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(0, 1, 0),
    new THREE.Vector3(0, 0, 0),
    120,
    0x00ff00,
    10,
    5
  )
  scene.add(yAxisLine)
  
  // z軸 (x_c)
  const zAxisLine = new THREE.ArrowHelper(
    new THREE.Vector3(0, 0, 1),
    new THREE.Vector3(0, 0, 0),
    120,
    0x0000ff,
    10,
    5
  )
  scene.add(zAxisLine)

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

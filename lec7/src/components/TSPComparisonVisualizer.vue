<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

interface Point {
  x: number
  y: number
}

const canvas2OptRef = ref<HTMLCanvasElement | null>(null)
const canvasSARef = ref<HTMLCanvasElement | null>(null)
const points = ref<Point[]>([])
const tour2Opt = ref<number[]>([])
const tourSA = ref<number[]>([])
const isRunning = ref(false)
const animationFrameId = ref<number | null>(null)
const temperatureSchedule = ref<'logarithmic' | 'linear'>('logarithmic') // 温度スケジューリングの選択
const numPoints = ref<number>(20) // 頂点数
const edgePairs2Opt = ref<[number, number][]>([]) // シャッフルされた辺のペアリスト
const edgePairsSA = ref<[number, number][]>([]) // シャッフルされた辺のペアリスト
let edgeIndex2Opt = 0 // 現在のインデックス
let edgeIndexSA = 0 // 現在のインデックス
const fps = 10 // 1秒に10回更新（高速化）
const frameInterval = 1000 / fps

const canvasWidth = 400
const canvasHeight = 300
const margin = 30

// 温度関数: T(t) = 10 / t
let iteration2Opt = 0
let iterationSA = 0

// ユークリッド距離を計算
function distance(p1: Point, p2: Point): number {
  return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2)
}

// 順回路の総距離を計算
function totalDistance(tour: number[]): number {
  if (tour.length < 2) return 0
  let dist = 0
  for (let i = 0; i < tour.length; i++) {
    const j = (i + 1) % tour.length
    dist += distance(points.value[tour[i]], points.value[tour[j]])
  }
  return dist
}

// すべての有効な辺のペアを生成してシャッフル
function generateEdgePairs(n: number): [number, number][] {
  const pairs: [number, number][] = []
  
  for (let i1 = 0; i1 < n; i1++) {
    for (let i2 = i1 + 2; i2 < n; i2++) {
      // 隣接している場合はスキップ
      if (i2 === i1 + 1 || (i1 === 0 && i2 === n - 1)) continue
      pairs.push([i1, i2])
    }
  }
  
  // Fisher-Yates shuffle
  for (let i = pairs.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[pairs[i], pairs[j]] = [pairs[j], pairs[i]]
  }
  
  return pairs
}

// 点を生成（均一に散らばらせる）
function generatePoints() {
  const n = numPoints.value
  const newPoints: Point[] = []
  
  // ランダムに均一に散らばらせる
  for (let i = 0; i < n; i++) {
    const x = margin + Math.random() * (canvasWidth - 2 * margin)
    const y = margin + Math.random() * (canvasHeight - 2 * margin)
    newPoints.push({ x, y })
  }
  
  points.value = newPoints
  
  // 同じランダム順回路を生成（両方のアルゴリズムで使用）
  const randomTour = Array.from({ length: n }, (_, i) => i)
  // Fisher-Yates shuffle
  for (let i = randomTour.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [randomTour[i], randomTour[j]] = [randomTour[j], randomTour[i]]
  }
  
  tour2Opt.value = [...randomTour]
  tourSA.value = [...randomTour]
  iteration2Opt = 0
  iterationSA = 0
  
  // 辺のペアを生成してシャッフル
  edgePairs2Opt.value = generateEdgePairs(n)
  edgePairsSA.value = generateEdgePairs(n)
  edgeIndex2Opt = 0
  edgeIndexSA = 0
}

// 2-opt交換を実行（改善がある場合のみ受容）
function perform2OptStep(): boolean {
  const n = tour2Opt.value.length
  
  // シャッフルされたリストから順番に選ぶ
  if (edgeIndex2Opt >= edgePairs2Opt.value.length) {
    // リストの最後まで行ったら、再度シャッフル
    edgePairs2Opt.value = generateEdgePairs(n)
    edgeIndex2Opt = 0
  }
  
  const [i1, i2] = edgePairs2Opt.value[edgeIndex2Opt]
  edgeIndex2Opt++
  
  // 現在の距離
  const [p1Idx1, p2Idx1] = [tour2Opt.value[i1], tour2Opt.value[(i1 + 1) % n]]
  const [p1Idx2, p2Idx2] = [tour2Opt.value[i2], tour2Opt.value[(i2 + 1) % n]]
  const currentDist = distance(points.value[p1Idx1], points.value[p2Idx1]) +
                      distance(points.value[p1Idx2], points.value[p2Idx2])
  
  // 交換後の距離
  const newDist = distance(points.value[p1Idx1], points.value[p1Idx2]) +
                 distance(points.value[p2Idx1], points.value[p2Idx2])
  
  const improvement = currentDist - newDist
  
  // 改善がある場合のみ受容（2-opt法は改善がある場合のみ更新）
  if (improvement > 0) {
    const e1 = i1
    const e2 = i2
    
    const newTour: number[] = []
    for (let i = 0; i <= e1; i++) {
      newTour.push(tour2Opt.value[i])
    }
    for (let i = e2; i > e1; i--) {
      newTour.push(tour2Opt.value[i])
    }
    for (let i = e2 + 1; i < n; i++) {
      newTour.push(tour2Opt.value[i])
    }
    
    if (newTour.length === n) {
      tour2Opt.value = newTour
      return true // 受容
    }
  }
  
  return false // 拒否
}

// 焼きなまし法の1ステップ
function performSAStep(): boolean {
  const n = tourSA.value.length
  // 温度関数の選択
  const T0 = 10
  let T: number
  if (temperatureSchedule.value === 'logarithmic') {
    // 対数スケール: T(t) = T_0 / log(2 + t)
    T = T0 / Math.log(2 + iterationSA)
  } else {
    // 線形スケール: T(t) = T_0 / t (t >= 1)
    T = iterationSA > 0 ? T0 / iterationSA : T0
  }
  
  // シャッフルされたリストから順番に選ぶ
  if (edgeIndexSA >= edgePairsSA.value.length) {
    // リストの最後まで行ったら、再度シャッフル
    edgePairsSA.value = generateEdgePairs(n)
    edgeIndexSA = 0
  }
  
  const [i1, i2] = edgePairsSA.value[edgeIndexSA]
  edgeIndexSA++
  
  // 現在の距離
  const [p1Idx1, p2Idx1] = [tourSA.value[i1], tourSA.value[(i1 + 1) % n]]
  const [p1Idx2, p2Idx2] = [tourSA.value[i2], tourSA.value[(i2 + 1) % n]]
  const currentDist = distance(points.value[p1Idx1], points.value[p2Idx1]) +
                      distance(points.value[p1Idx2], points.value[p2Idx2])
  
  // 交換後の距離
  const newDist = distance(points.value[p1Idx1], points.value[p1Idx2]) +
                 distance(points.value[p2Idx1], points.value[p2Idx2])
  
  const deltaW = newDist - currentDist
  
  // 更新確率を計算
  let accept = false
  if (deltaW < 0) {
    // 改善する場合は必ず受け入れる
    accept = true
  } else {
    // 悪化する場合は確率的に受け入れる
    const theta = Math.exp(-deltaW / T)
    accept = Math.random() < theta
  }
  
  if (accept) {
    const e1 = i1
    const e2 = i2
    
    const newTour: number[] = []
    for (let i = 0; i <= e1; i++) {
      newTour.push(tourSA.value[i])
    }
    for (let i = e2; i > e1; i--) {
      newTour.push(tourSA.value[i])
    }
    for (let i = e2 + 1; i < n; i++) {
      newTour.push(tourSA.value[i])
    }
    
    if (newTour.length === n) {
      tourSA.value = newTour
      return true
    }
  }
  
  return false
}

// 描画（2-opt用）
function draw2Opt() {
  if (!canvas2OptRef.value) return
  
  const ctx = canvas2OptRef.value.getContext('2d')
  if (!ctx) return
  
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  // 順回路を描画
  if (tour2Opt.value.length > 1) {
    ctx.strokeStyle = '#666'
    ctx.lineWidth = 1
    ctx.beginPath()
    
    for (let i = 0; i < tour2Opt.value.length; i++) {
      const p = points.value[tour2Opt.value[i]]
      if (i === 0) {
        ctx.moveTo(p.x, p.y)
      } else {
        ctx.lineTo(p.x, p.y)
      }
    }
    ctx.closePath()
    ctx.stroke()
  }
  
  // 点を描画
  ctx.fillStyle = '#1976d2'
  for (const point of points.value) {
    ctx.beginPath()
    ctx.arc(point.x, point.y, 4, 0, 2 * Math.PI)
    ctx.fill()
  }
  
  // 目的関数値と反復回数を表示（左上）
  ctx.fillStyle = '#000'
  ctx.font = 'bold 14px sans-serif'
  const distanceText = `距離: ${totalDistance(tour2Opt.value).toFixed(2)}`
  const iterationText = `反復回数: ${iteration2Opt}`
  ctx.fillText(distanceText, 10, 20)
  ctx.font = '12px sans-serif'
  const distanceWidth = ctx.measureText(distanceText).width
  ctx.fillText(iterationText, 10 + distanceWidth + 30, 20)
}

// 描画（焼きなまし法用）
function drawSA() {
  if (!canvasSARef.value) return
  
  const ctx = canvasSARef.value.getContext('2d')
  if (!ctx) return
  
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  // 順回路を描画
  if (tourSA.value.length > 1) {
    ctx.strokeStyle = '#666'
    ctx.lineWidth = 1
    ctx.beginPath()
    
    for (let i = 0; i < tourSA.value.length; i++) {
      const p = points.value[tourSA.value[i]]
      if (i === 0) {
        ctx.moveTo(p.x, p.y)
      } else {
        ctx.lineTo(p.x, p.y)
      }
    }
    ctx.closePath()
    ctx.stroke()
  }
  
  // 点を描画
  ctx.fillStyle = '#1976d2'
  for (const point of points.value) {
    ctx.beginPath()
    ctx.arc(point.x, point.y, 4, 0, 2 * Math.PI)
    ctx.fill()
  }
  
  // 目的関数値と反復回数を表示（左上）
  ctx.fillStyle = '#000'
  ctx.font = 'bold 14px sans-serif'
  const distanceText = `距離: ${totalDistance(tourSA.value).toFixed(2)}`
  const iterationText = `反復回数: ${iterationSA}`
  ctx.fillText(distanceText, 10, 20)
  ctx.font = '12px sans-serif'
  const distanceWidth = ctx.measureText(distanceText).width
  ctx.fillText(iterationText, 10 + distanceWidth + 30, 20)
}

// アニメーションループ
let lastUpdateTime = 0
function animate(currentTime: number) {
  if (!isRunning.value) return
  
  if (currentTime - lastUpdateTime >= frameInterval) {
    // 2-opt法のステップ（判定後に時刻を増やす）
    perform2OptStep()
    iteration2Opt++
    
    // 焼きなまし法のステップ（判定後に時刻を増やす）
    performSAStep()
    iterationSA++
    
    // 描画
    draw2Opt()
    drawSA()
    
    lastUpdateTime = currentTime
  }
  
  animationFrameId.value = requestAnimationFrame(animate)
}

// 開始/停止
function toggleAnimation() {
  if (isRunning.value) {
    // 停止
    if (animationFrameId.value !== null) {
      cancelAnimationFrame(animationFrameId.value)
      animationFrameId.value = null
    }
    isRunning.value = false
  } else {
    // 開始
    isRunning.value = true
    lastUpdateTime = performance.now()
    animationFrameId.value = requestAnimationFrame(animate)
  }
}

// リセット
function reset() {
  if (isRunning.value) {
    toggleAnimation()
  }
  generatePoints()
  draw2Opt()
  drawSA()
}

// 頂点数が変更されたときに再生成
watch(numPoints, () => {
  if (!isRunning.value) {
    generatePoints()
    draw2Opt()
    drawSA()
  }
})

onMounted(() => {
  generatePoints()
  draw2Opt()
  drawSA()
})

onUnmounted(() => {
  if (animationFrameId.value !== null) {
    cancelAnimationFrame(animationFrameId.value)
  }
})
</script>

<template>
  <div style="display: flex; flex-direction: column; align-items: center; gap: 1em;">
    <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em;">
      <div style="display: flex; flex-direction: column; align-items: center; gap: 0.5em;">
        <div style="font-weight: bold; font-size: 16px;">2-opt法</div>
        <canvas
          ref="canvas2OptRef"
          :width="canvasWidth"
          :height="canvasHeight"
          style="border: 1px solid #ccc; border-radius: 8px; display: block;"
        ></canvas>
      </div>
      <div style="display: flex; flex-direction: column; align-items: center; gap: 0.5em;">
        <div style="font-weight: bold; font-size: 16px;">焼きなまし法</div>
        <canvas
          ref="canvasSARef"
          :width="canvasWidth"
          :height="canvasHeight"
          style="border: 1px solid #ccc; border-radius: 8px; display: block;"
        ></canvas>
      </div>
    </div>
    <div style="display: flex; flex-direction: column; gap: 1em; align-items: center;">
      <div style="display: flex; flex-wrap: wrap; gap: 1em; justify-content: center; align-items: center;">
        <div style="display: flex; align-items: center; gap: 1em;">
          <label style="font-size: 14px; font-weight: bold;">
            頂点数:
          </label>
          <select
            v-model.number="numPoints"
            :disabled="isRunning"
            style="padding: 5px 10px; font-size: 14px; border: 1px solid #ccc; border-radius: 4px;"
          >
            <option :value="10">10</option>
            <option :value="20">20</option>
            <option :value="30">30</option>
            <option :value="40">40</option>
            <option :value="50">50</option>
          </select>
        </div>
        <div style="display: flex; align-items: center; gap: 1em;">
          <label style="font-size: 14px; font-weight: bold;">
            温度スケジューリング:
          </label>
          <select
            v-model="temperatureSchedule"
            :disabled="isRunning"
            style="padding: 5px 10px; font-size: 14px; border: 1px solid #ccc; border-radius: 4px;"
          >
            <option value="logarithmic">対数スケール (T(t) = T₀ / log(2+t))</option>
            <option value="linear">線形スケール (T(t) = T₀ / t)</option>
          </select>
        </div>
      </div>
      <div style="display: flex; gap: 1em;">
        <button
          @click="toggleAnimation"
          :style="{ padding: '10px 20px', background: isRunning ? '#d32f2f' : '#388e3c', color: 'white', border: 'none', borderRadius: '4px', cursor: 'pointer', fontSize: '14px', fontWeight: 'bold' }"
        >
          {{ isRunning ? '停止' : '開始' }}
        </button>
        <button
          @click="reset"
          :disabled="isRunning"
          style="padding: 10px 20px; background: #666; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 14px;"
          :style="{ opacity: isRunning ? 0.5 : 1, cursor: isRunning ? 'not-allowed' : 'pointer' }"
        >
          リセット
        </button>
      </div>
    </div>
  </div>
</template>


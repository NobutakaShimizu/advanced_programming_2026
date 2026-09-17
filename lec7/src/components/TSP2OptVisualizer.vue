<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

interface Point {
  x: number
  y: number
}

const canvasRef = ref<HTMLCanvasElement | null>(null)
const points = ref<Point[]>([])
const tour = ref<number[]>([])
const isRunning = ref(false)
const animationFrameId = ref<number | null>(null)
const numPoints = ref<number>(20) // 頂点数
const edgePairs = ref<[number, number][]>([]) // 辺のペアリスト
let edgeIndex = 0 // 現在のインデックス
let lastImprovementIndex: number | null = null // 最後に改善が見つかった位置
let hasImprovementInCycle = false // 現在の周回で改善があったか
const fps = 20 // 1秒に20回更新
const frameInterval = 1000 / fps

const canvasWidth = 600
const canvasHeight = 400
const margin = 30

let iteration = 0

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

// すべての有効な辺のペアを生成（順番に走査）
function generateEdgePairs(n: number): [number, number][] {
  const pairs: [number, number][] = []
  
  for (let i1 = 0; i1 < n; i1++) {
    for (let i2 = i1 + 2; i2 < n; i2++) {
      // 隣接している場合はスキップ
      if (i2 === i1 + 1 || (i1 === 0 && i2 === n - 1)) continue
      pairs.push([i1, i2])
    }
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
  
  // ランダム順回路を生成
  const randomTour = Array.from({ length: n }, (_, i) => i)
  // Fisher-Yates shuffle
  for (let i = randomTour.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [randomTour[i], randomTour[j]] = [randomTour[j], randomTour[i]]
  }
  
  tour.value = [...randomTour]
  iteration = 0
  
  // 辺のペアを生成（順番に走査）
  edgePairs.value = generateEdgePairs(n)
  edgeIndex = 0
  lastImprovementIndex = null
  hasImprovementInCycle = false
}

// 2-opt交換を実行（改善がある場合のみ受容）
// 戻り値: true=改善があった, false=改善なし, null=最後の改善位置に戻って改善なし（停止すべき）
function perform2OptStep(): boolean | null {
  const n = tour.value.length
  
  // 最後の改善位置に戻ったかチェック（最初の改善が見つかる前はnullなのでスキップ）
  if (lastImprovementIndex !== null && edgeIndex === lastImprovementIndex) {
    // 最後の改善位置に戻った：その間改善がなかった場合は停止
    if (!hasImprovementInCycle) {
      return null // 停止を促す
    }
    // 改善があった場合は次の周回を開始
    hasImprovementInCycle = false
  }
  
  // 順番に走査（最後まで行ったら最初に戻る）
  if (edgeIndex >= edgePairs.value.length) {
    // 最初の改善が見つかる前の場合は、全周回をチェック
    if (lastImprovementIndex === null) {
      // 1周完了：改善がなかった場合は停止を促す
      if (!hasImprovementInCycle) {
        edgeIndex = 0 // リセット
        return null // 1周完了して改善なし、停止を促す
      }
      // 改善があった場合は次の周回を開始
      edgeIndex = 0
      hasImprovementInCycle = false
    } else {
      // 既に改善が見つかっている場合は、循環して続ける
      edgeIndex = 0
    }
  }
  
  const [i1, i2] = edgePairs.value[edgeIndex]
  const currentEdgeIndex = edgeIndex // 現在のインデックスを保存
  edgeIndex++
  
  // 現在の距離
  const [p1Idx1, p2Idx1] = [tour.value[i1], tour.value[(i1 + 1) % n]]
  const [p1Idx2, p2Idx2] = [tour.value[i2], tour.value[(i2 + 1) % n]]
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
      newTour.push(tour.value[i])
    }
    for (let i = e2; i > e1; i--) {
      newTour.push(tour.value[i])
    }
    for (let i = e2 + 1; i < n; i++) {
      newTour.push(tour.value[i])
    }
    
    if (newTour.length === n) {
      tour.value = newTour
      hasImprovementInCycle = true
      // 改善が見つかった位置を記録（次の探索の開始位置）
      lastImprovementIndex = currentEdgeIndex
      return true // 受容
    }
  }
  
  return false // 改善なし（続行）
}

// 描画
function draw() {
  if (!canvasRef.value) return
  
  const ctx = canvasRef.value.getContext('2d')
  if (!ctx) return
  
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  // 順回路を描画
  if (tour.value.length > 1) {
    ctx.strokeStyle = '#666'
    ctx.lineWidth = 1.5
    ctx.beginPath()
    
    for (let i = 0; i < tour.value.length; i++) {
      const p = points.value[tour.value[i]]
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
    ctx.arc(point.x, point.y, 5, 0, 2 * Math.PI)
    ctx.fill()
  }
  
  // 目的関数値と反復回数を表示（左上）
  ctx.fillStyle = '#000'
  ctx.font = 'bold 16px sans-serif'
  const distanceText = `距離: ${totalDistance(tour.value).toFixed(2)}`
  const iterationText = `反復回数: ${iteration}`
  ctx.fillText(distanceText, 15, 25)
  ctx.font = '14px sans-serif'
  const distanceWidth = ctx.measureText(distanceText).width
  ctx.fillText(iterationText, 15 + distanceWidth + 30, 25)
}

// アニメーションループ
let lastUpdateTime = 0
function animate(currentTime: number) {
  if (!isRunning.value) return
  
  if (currentTime - lastUpdateTime >= frameInterval) {
    // 2-opt法のステップ（判定後に時刻を増やす）
    const result = perform2OptStep()
    iteration++
    
    // 全ての辺のペアをチェックしても改善がなかった場合は停止
    if (result === null) {
      isRunning.value = false
      if (animationFrameId.value !== null) {
        cancelAnimationFrame(animationFrameId.value)
        animationFrameId.value = null
      }
    }
    
    // 描画
    draw()
    
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
  draw()
}

// 頂点数が変更されたときに再生成
watch(numPoints, () => {
  if (!isRunning.value) {
    generatePoints()
    draw()
  }
})

onMounted(() => {
  generatePoints()
  draw()
})

onUnmounted(() => {
  if (animationFrameId.value !== null) {
    cancelAnimationFrame(animationFrameId.value)
  }
})
</script>

<template>
  <div style="display: flex; flex-direction: column; align-items: center; gap: 1em;">
    <div style="position: relative; display: inline-block;">
      <canvas
        ref="canvasRef"
        :width="canvasWidth"
        :height="canvasHeight"
        style="border: 1px solid #ccc; border-radius: 8px; display: block;"
      ></canvas>
      <div style="position: absolute; top: 10px; right: 10px; display: flex; flex-wrap: wrap; gap: 0.5em; justify-content: flex-end; align-items: center; z-index: 10;">
        <div style="display: flex; align-items: center; gap: 0.5em; background: rgba(255, 255, 255, 0.9); padding: 5px 10px; border-radius: 4px;">
          <label style="font-size: 12px; font-weight: bold;">
            頂点数:
          </label>
          <select
            v-model.number="numPoints"
            :disabled="isRunning"
            style="padding: 3px 8px; font-size: 12px; border: 1px solid #ccc; border-radius: 4px;"
          >
            <option :value="10">10</option>
            <option :value="20">20</option>
            <option :value="30">30</option>
            <option :value="40">40</option>
            <option :value="50">50</option>
          </select>
        </div>
        <button
          @click="toggleAnimation"
          :style="{ padding: '5px 15px', background: isRunning ? '#d32f2f' : '#388e3c', color: 'white', border: 'none', borderRadius: '4px', cursor: 'pointer', fontSize: '12px', fontWeight: 'bold' }"
        >
          {{ isRunning ? '停止' : '開始' }}
        </button>
        <button
          @click="reset"
          :disabled="isRunning"
          :style="{ padding: '5px 15px', background: '#666', color: 'white', border: 'none', borderRadius: '4px', cursor: isRunning ? 'not-allowed' : 'pointer', fontSize: '12px', opacity: isRunning ? 0.5 : 1 }"
        >
          リセット
        </button>
      </div>
    </div>
  </div>
</template>


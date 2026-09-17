<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const canvasLeftRef = ref<HTMLCanvasElement | null>(null)
const canvasRightRef = ref<HTMLCanvasElement | null>(null)
const isRunning = ref(false)
const animationTimer = ref<number | null>(null)

const n = 15 // 頂点数
const canvasWidth = 400
const canvasHeight = 400
const margin = 50

// グラフ構造（隣接リスト）
let graph: number[][] = []
// 頂点の位置（円形に配置）
let vertexPositions: { x: number; y: number }[] = []
// 現在の頂点
let currentVertex = 0
// 訪問回数カウント
let visitCounts = new Array(n).fill(0)
// 現在の時刻
let t = 0

// 連結グラフを生成（適当に）
function generateGraph() {
  // 各頂点をランダムに配置
  vertexPositions = []
  for (let i = 0; i < n; i++) {
    vertexPositions.push({
      x: margin + Math.random() * (canvasWidth - 2 * margin),
      y: margin + Math.random() * (canvasHeight - 2 * margin)
    })
  }
  
  // グラフを生成（完全なランダムグラフ、次数にばらつきあり）
  graph = []
  for (let i = 0; i < n; i++) {
    graph[i] = []
  }
  
  // 各頂点に「接続しやすさ」をランダムに割り当て（次数のばらつきを作る）
  const connectionProbabilities: number[] = []
  for (let i = 0; i < n; i++) {
    // 0.05から0.25の間でランダムに設定（sparseで次数にばらつきがある）
    // より広い範囲で設定することで、次数のばらつきを増やす
    connectionProbabilities.push(0.05 + Math.random() * 0.2)
  }
  
  // まず、最小全域木を作って連結性を保証
  const parent: number[] = []
  for (let i = 0; i < n; i++) {
    parent[i] = i
  }
  
  function find(x: number): number {
    if (parent[x] !== x) {
      parent[x] = find(parent[x])
    }
    return parent[x]
  }
  
  function union(x: number, y: number): boolean {
    const px = find(x)
    const py = find(y)
    if (px === py) return false
    parent[px] = py
    return true
  }
  
  // すべての辺の候補を生成してシャッフル
  const allEdges: [number, number][] = []
  for (let i = 0; i < n; i++) {
    for (let j = i + 1; j < n; j++) {
      allEdges.push([i, j])
    }
  }
  
  // Fisher-Yates shuffle
  for (let i = allEdges.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[allEdges[i], allEdges[j]] = [allEdges[j], allEdges[i]]
  }
  
  // 最小全域木を構築（連結性を保証）
  let mstEdges = 0
  for (const [u, v] of allEdges) {
    if (union(u, v)) {
      graph[u].push(v)
      graph[v].push(u)
      mstEdges++
      if (mstEdges === n - 1) break
    }
  }
  
  // 残りの辺をランダムに追加（完全なランダムグラフ）
  for (const [u, v] of allEdges) {
    // 既に辺が存在する場合はスキップ
    if (graph[u].includes(v)) continue
    
    // 両方の頂点の接続しやすさの平均を確率として使用
    const prob = (connectionProbabilities[u] + connectionProbabilities[v]) / 2
    if (Math.random() < prob) {
      graph[u].push(v)
      graph[v].push(u)
    }
  }
}

// ランダムウォークの1ステップ
function randomWalkStep() {
  // 自己ループ確率0（常に隣接頂点に移動）
  const neighbors = graph[currentVertex]
  if (neighbors.length > 0) {
    const nextIndex = Math.floor(Math.random() * neighbors.length)
    currentVertex = neighbors[nextIndex]
    visitCounts[currentVertex]++
  }
  t++
}

// 左半分の描画（グラフとランダムウォーク）
function drawLeft() {
  if (!canvasLeftRef.value) return
  
  const ctx = canvasLeftRef.value.getContext('2d')
  if (!ctx) return
  
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  // 辺を描画
  ctx.strokeStyle = '#000'
  ctx.lineWidth = 1
  for (let i = 0; i < n; i++) {
    for (const j of graph[i]) {
      if (i < j) { // 重複を避ける
        const p1 = vertexPositions[i]
        const p2 = vertexPositions[j]
        ctx.beginPath()
        ctx.moveTo(p1.x, p1.y)
        ctx.lineTo(p2.x, p2.y)
        ctx.stroke()
      }
    }
  }
  
  // 頂点を描画
  for (let i = 0; i < n; i++) {
    const pos = vertexPositions[i]
    ctx.fillStyle = i === currentVertex ? '#d32f2f' : '#1976d2'
    ctx.beginPath()
    ctx.arc(pos.x, pos.y, i === currentVertex ? 10 : 6, 0, 2 * Math.PI)
    ctx.fill()
    
    // 頂点番号を表示
    ctx.fillStyle = '#000'
    ctx.font = '12px sans-serif'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.fillText((i + 1).toString(), pos.x, pos.y - 18)
  }
  
  // 時刻を表示
  ctx.fillStyle = '#000'
  ctx.font = 'bold 14px sans-serif'
  ctx.textAlign = 'left'
  ctx.textBaseline = 'top'
  ctx.fillText(`時刻: ${t}`, 10, 10)
}

// 右半分の描画（μ_t(i)のプロット）
function drawRight() {
  if (!canvasRightRef.value) return
  
  const ctx = canvasRightRef.value.getContext('2d')
  if (!ctx) return
  
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  const plotWidth = canvasWidth - 2 * margin
  const plotHeight = canvasHeight - 2 * margin
  
  // μ_t(i)を計算
  const mu = new Array(n)
  if (t > 0) {
    for (let i = 0; i < n; i++) {
      mu[i] = visitCounts[i] / t
    }
  } else {
    for (let i = 0; i < n; i++) {
      mu[i] = 0
    }
  }
  
  // 縦軸の範囲は[0,1]で固定
  const yMin = 0
  const yMax = 1
  
  // グリッドを描画
  ctx.strokeStyle = '#ddd'
  ctx.lineWidth = 1
  for (let i = 0; i <= 5; i++) {
    const y = margin + (i / 5) * plotHeight
    ctx.beginPath()
    ctx.moveTo(margin, y)
    ctx.lineTo(margin + plotWidth, y)
    ctx.stroke()
  }
  
  // 棒グラフを描画
  const barWidth = plotWidth / n
  for (let i = 0; i < n; i++) {
    // [0,1]の範囲で正規化
    const normalizedMu = Math.max(0, Math.min(1, mu[i])) // 念のため[0,1]にクランプ
    const barHeight = normalizedMu * plotHeight
    const x = margin + i * barWidth
    const y = margin + plotHeight - barHeight
    
    ctx.fillStyle = '#1976d2'
    ctx.fillRect(x, y, barWidth * 0.8, barHeight)
    
    // 頂点番号のラベル
    ctx.fillStyle = '#000'
    ctx.font = '10px sans-serif'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'top'
    ctx.fillText((i + 1).toString(), x + barWidth * 0.4, margin + plotHeight + 5)
  }
  
  // 軸ラベル
  ctx.fillStyle = '#000'
  ctx.font = 'bold 14px sans-serif'
  ctx.textAlign = 'center'
  ctx.fillText('頂点番号', canvasWidth / 2, canvasHeight - 10)
  ctx.save()
  ctx.translate(15, canvasHeight / 2)
  ctx.rotate(-Math.PI / 2)
  ctx.fillText('μ_t(i)', 0, 0)
  ctx.restore()
  
  // 時刻を表示
  ctx.fillStyle = '#000'
  ctx.font = 'bold 14px sans-serif'
  ctx.textAlign = 'left'
  ctx.textBaseline = 'top'
  ctx.fillText(`時刻: ${t}`, 10, 10)
}

// アニメーションループ（1秒に4フレーム）
function animate() {
  if (!isRunning.value) return
  
  randomWalkStep()
  drawLeft()
  drawRight()
  
  // 1秒に4フレーム（250ms）後に次のフレーム
  animationTimer.value = window.setTimeout(animate, 250)
}

// 再生/停止
function toggleAnimation() {
  if (isRunning.value) {
    // 停止
    if (animationTimer.value !== null) {
      clearTimeout(animationTimer.value)
      animationTimer.value = null
    }
    isRunning.value = false
  } else {
    // 再生
    isRunning.value = true
    animate()
  }
}

// リセット
function reset() {
  if (isRunning.value) {
    toggleAnimation()
  }
  t = 0
  currentVertex = 0
  visitCounts = new Array(n).fill(0)
  generateGraph()
  drawLeft()
  drawRight()
}

onMounted(() => {
  generateGraph()
  drawLeft()
  drawRight()
})

onUnmounted(() => {
  if (animationTimer.value !== null) {
    clearTimeout(animationTimer.value)
  }
})
</script>

<template>
  <div style="display: flex; flex-direction: column; align-items: center; gap: 1em;">
    <!-- 左右分割: グラフとランダムウォーク（左）、μ_t(i)のプロット（右） -->
    <div style="display: flex; flex-direction: row; align-items: center; gap: 1em;">
      <div style="display: flex; flex-direction: column; align-items: center;">
        <canvas
          ref="canvasLeftRef"
          :width="canvasWidth"
          :height="canvasHeight"
          style="border: 1px solid #ccc; border-radius: 8px; display: block;"
        ></canvas>
      </div>
      
      <div style="display: flex; flex-direction: column; align-items: center;">
        <canvas
          ref="canvasRightRef"
          :width="canvasWidth"
          :height="canvasHeight"
          style="border: 1px solid #ccc; border-radius: 8px; display: block;"
        ></canvas>
      </div>
    </div>
    
    <!-- コントロールボタン -->
    <div style="display: flex; gap: 1em;">
      <button
        @click="toggleAnimation"
        :style="{ 
          padding: '10px 20px', 
          background: isRunning ? '#d32f2f' : '#388e3c', 
          color: 'white', 
          border: 'none', 
          borderRadius: '4px', 
          cursor: 'pointer', 
          fontSize: '14px', 
          fontWeight: 'bold' 
        }"
      >
        {{ isRunning ? '停止' : '再生' }}
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
</template>


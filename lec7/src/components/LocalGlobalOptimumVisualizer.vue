<script setup lang="ts">
import { ref, onMounted } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)
const canvasWidth = 450
const canvasHeight = 300
const margin = 50

// 関数: 複数の極値を持つ関数（最小化問題）
function f(x: number): number {
  // 複数の極値を持つ関数（例: sin波と多項式の組み合わせ）
  // 両端が上向きになるように2次項を追加
  const center = 5 // 中央
  const quadratic = 0.15 * (x - center) * (x - center) // 両端が上がる2次項
  return 2 * Math.sin(x * 0.5) + 0.5 * Math.sin(x * 2) + quadratic + 3
}

// 描画
function draw() {
  if (!canvasRef.value) return
  
  const ctx = canvasRef.value.getContext('2d')
  if (!ctx) return
  
  // クリア
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  
  // 背景
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  const plotWidth = canvasWidth - 2 * margin
  const plotHeight = canvasHeight - 2 * margin
  const xMin = 0
  const xMax = 10
  
  // yの範囲を動的に計算
  let yMin = Infinity
  let yMax = -Infinity
  for (let x = xMin; x <= xMax; x += 0.1) {
    const y = f(x)
    if (y < yMin) yMin = y
    if (y > yMax) yMax = y
  }
  // 少し余白を追加
  const yRange = yMax - yMin
  yMin -= yRange * 0.1
  yMax += yRange * 0.1
  
  // グリッドを描画
  ctx.strokeStyle = '#ddd'
  ctx.lineWidth = 1
  for (let i = 0; i <= 10; i++) {
    const x = margin + (i / 10) * plotWidth
    ctx.beginPath()
    ctx.moveTo(x, margin)
    ctx.lineTo(x, margin + plotHeight)
    ctx.stroke()
    
    const y = margin + (i / 10) * plotHeight
    ctx.beginPath()
    ctx.moveTo(margin, y)
    ctx.lineTo(margin + plotWidth, y)
    ctx.stroke()
  }
  
  // 関数を描画
  ctx.strokeStyle = '#1976d2'
  ctx.lineWidth = 3
  ctx.beginPath()
  
  const step = 0.1
  for (let x = xMin; x <= xMax; x += step) {
    const y = f(x)
    const plotX = margin + ((x - xMin) / (xMax - xMin)) * plotWidth
    const plotY = margin + plotHeight - ((y - yMin) / (yMax - yMin)) * plotHeight
    
    if (x === xMin) {
      ctx.moveTo(plotX, plotY)
    } else {
      ctx.lineTo(plotX, plotY)
    }
  }
  ctx.stroke()
  
  // 局所最適解と大域最適解を探す
  const localOptima: { x: number; y: number; isGlobal: boolean }[] = []
  
  // サンプルポイントで極値を探す
  for (let x = xMin + 0.5; x < xMax - 0.5; x += 0.1) {
    const y = f(x)
    const yPrev = f(x - 0.1)
    const yNext = f(x + 0.1)
    
    // 極小値（下に凸）
    if (y < yPrev && y < yNext) {
      localOptima.push({ x, y, isGlobal: false })
    }
  }
  
  // 大域最適解を見つける（最小値）
  let globalMin = Infinity
  let globalMinX = 0
  for (const opt of localOptima) {
    if (opt.y < globalMin) {
      globalMin = opt.y
      globalMinX = opt.x
    }
  }
  
  // 局所最適解を描画（赤）
  for (const opt of localOptima) {
    const plotX = margin + ((opt.x - xMin) / (xMax - xMin)) * plotWidth
    const plotY = margin + plotHeight - ((opt.y - yMin) / (yMax - yMin)) * plotHeight
    
    if (Math.abs(opt.x - globalMinX) < 0.2) {
      // 大域最適解（緑）
      ctx.fillStyle = '#388e3c'
      ctx.strokeStyle = '#2e7d32'
      ctx.lineWidth = 3
    } else {
      // 局所最適解（赤）
      ctx.fillStyle = '#d32f2f'
      ctx.strokeStyle = '#c62828'
      ctx.lineWidth = 2
    }
    
    // 点を描画
    ctx.beginPath()
    ctx.arc(plotX, plotY, 8, 0, 2 * Math.PI)
    ctx.fill()
    ctx.stroke()
    
    // ラベル
    ctx.fillStyle = '#000'
    ctx.font = 'bold 12px sans-serif'
    if (Math.abs(opt.x - globalMinX) < 0.2) {
      ctx.fillText('大域最適解', plotX + 15, plotY - 15)
    } else {
      ctx.fillText('局所最適解', plotX + 15, plotY - 15)
    }
  }
  
  // 軸ラベル
  ctx.fillStyle = '#000'
  ctx.font = '14px sans-serif'
  ctx.fillText('x', canvasWidth - margin + 10, canvasHeight - margin + 5)
  ctx.fillText('f(x)', margin - 30, margin - 10)
  
  // 説明テキスト
  ctx.fillStyle = '#666'
  ctx.font = '12px sans-serif'
  ctx.fillText('目的関数', margin, margin - 25)
}

onMounted(() => {
  draw()
})
</script>

<template>
  <div style="display: flex; justify-content: center;">
    <canvas
      ref="canvasRef"
      :width="canvasWidth"
      :height="canvasHeight"
      style="border: 1px solid #ccc; border-radius: 8px; display: block;"
    ></canvas>
  </div>
</template>


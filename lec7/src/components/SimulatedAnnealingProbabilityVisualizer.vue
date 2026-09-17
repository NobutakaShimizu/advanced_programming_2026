<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)
const deltaW = ref(1) // w(P') - w(P) の固定値
const canvasWidth = 500
const canvasHeight = 350
const margin = 50

// 更新確率を計算: θ = exp(-Δw / T)
function acceptanceProbability(T: number, delta: number): number {
  if (T <= 0) return 0
  return Math.exp(-delta / T)
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
  const TMin = 0.1
  const TMax = 10
  const thetaMin = 0
  const thetaMax = 1
  
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
  
  const step = 0.05
  for (let T = TMin; T <= TMax; T += step) {
    const theta = acceptanceProbability(T, deltaW.value)
    const plotX = margin + ((T - TMin) / (TMax - TMin)) * plotWidth
    const plotY = margin + plotHeight - ((theta - thetaMin) / (thetaMax - thetaMin)) * plotHeight
    
    if (T === TMin) {
      ctx.moveTo(plotX, plotY)
    } else {
      ctx.lineTo(plotX, plotY)
    }
  }
  ctx.stroke()
  
  // 軸ラベル
  ctx.fillStyle = '#000'
  ctx.font = 'bold 14px sans-serif'
  ctx.fillText('T', canvasWidth - margin + 10, canvasHeight - margin + 5)
  ctx.save()
  ctx.translate(margin - 30, canvasHeight / 2)
  ctx.rotate(-Math.PI / 2)
  ctx.fillText('θ', 0, 0)
  ctx.restore()
  
  // 軸の目盛り
  ctx.fillStyle = '#666'
  ctx.font = '12px sans-serif'
  
  // T軸の目盛り
  for (let i = 0; i <= 5; i++) {
    const T = TMin + (i / 5) * (TMax - TMin)
    const x = margin + ((T - TMin) / (TMax - TMin)) * plotWidth
    ctx.fillText(T.toFixed(1), x - 10, canvasHeight - margin + 20)
  }
  
  // θ軸の目盛り
  for (let i = 0; i <= 5; i++) {
    const theta = thetaMin + (i / 5) * (thetaMax - thetaMin)
    const y = margin + plotHeight - ((theta - thetaMin) / (thetaMax - thetaMin)) * plotHeight
    ctx.fillText(theta.toFixed(1), margin - 35, y + 4)
  }
  
  // パラメータ表示
  ctx.fillStyle = '#d32f2f'
  ctx.font = 'bold 12px sans-serif'
  ctx.fillText(`Δw = w(P') - w(P) = ${deltaW.value}`, margin, margin - 10)
  
  // 説明テキスト
  ctx.fillStyle = '#666'
  ctx.font = '12px sans-serif'
  ctx.fillText('θ = exp(-Δw / T)', margin, margin - 30)
}

// deltaWが変更されたときに再描画
const deltaWDisplay = computed(() => deltaW.value)

onMounted(() => {
  draw()
})

// deltaWの変更を監視
watch(deltaW, () => {
  draw()
})
</script>

<template>
  <div style="display: flex; flex-direction: column; align-items: center; gap: 1em;">
    <div style="display: flex; align-items: center; gap: 1em;">
      <label style="font-size: 14px; font-weight: bold;">
        Δw = w(P') - w(P):
      </label>
      <input
        v-model.number="deltaW"
        type="range"
        min="0.1"
        max="5"
        step="0.1"
        style="width: 200px;"
      />
      <span style="font-size: 14px; min-width: 50px; text-align: right;">
        {{ deltaW.toFixed(1) }}
      </span>
    </div>
    <canvas
      ref="canvasRef"
      :width="canvasWidth"
      :height="canvasHeight"
      style="border: 1px solid #ccc; border-radius: 8px; display: block;"
    ></canvas>
  </div>
</template>


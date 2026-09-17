<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const temperature = ref(25.0)
const canvasWidth = 600
const canvasHeight = 300
const margin = { top: 40, right: 40, bottom: 60, left: 60 }

// 適当にw(P)を設定（10個の解）
const nSolutions = 10
const wValues = ref<number[]>([])

// w(P)を初期化（ランダムな値）
function initializeWValues() {
  const values: number[] = []
  for (let i = 0; i < nSolutions; i++) {
    // 0から20の範囲でランダムに生成
    values.push(Math.random() * 20)
  }
  wValues.value = values
}

// ボルツマン分布を計算
const boltzmannDistribution = computed(() => {
  const Z = wValues.value.reduce((sum, w) => sum + Math.exp(-w / temperature.value), 0)
  return wValues.value.map(w => Math.exp(-w / temperature.value) / Z)
})

// 最小のw(P)を見つける
const minWIndex = computed(() => {
  let minIndex = 0
  let minValue = wValues.value[0]
  for (let i = 1; i < wValues.value.length; i++) {
    if (wValues.value[i] < minValue) {
      minValue = wValues.value[i]
      minIndex = i
    }
  }
  return minIndex
})

// T -> 0のときのDirac測度（最小w(P)の解だけが確率1）
const diracDistribution = computed(() => {
  const dist = new Array(nSolutions).fill(0)
  dist[minWIndex.value] = 1
  return dist
})

// 描画用のref
const canvasRef = ref<HTMLCanvasElement | null>(null)

function draw() {
  if (!canvasRef.value) return
  
  const ctx = canvasRef.value.getContext('2d')
  if (!ctx) return
  
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  
  // 背景
  ctx.fillStyle = '#f5f5f5'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)
  
  // 描画領域
  const plotWidth = canvasWidth - margin.left - margin.right
  const plotHeight = canvasHeight - margin.top - margin.bottom
  
  // Tが非常に小さい場合はDirac測度を表示
  const useDirac = temperature.value < 0.01
  const distribution = useDirac ? diracDistribution.value : boltzmannDistribution.value
  
  // 最大確率を見つける（スケーリング用）
  const maxProb = Math.max(...distribution)
  
  // グリッド線を描画
  ctx.strokeStyle = '#ddd'
  ctx.lineWidth = 1
  for (let i = 0; i <= 5; i++) {
    const y = margin.top + (plotHeight / 5) * i
    ctx.beginPath()
    ctx.moveTo(margin.left, y)
    ctx.lineTo(margin.left + plotWidth, y)
    ctx.stroke()
  }
  
  // 軸を描画
  ctx.strokeStyle = '#333'
  ctx.lineWidth = 2
  ctx.beginPath()
  ctx.moveTo(margin.left, margin.top)
  ctx.lineTo(margin.left, margin.top + plotHeight)
  ctx.lineTo(margin.left + plotWidth, margin.top + plotHeight)
  ctx.stroke()
  
  // 確率分布を描画（Pのインデックスを横軸、π(P)を縦軸に）
  ctx.strokeStyle = '#1976d2'
  ctx.fillStyle = '#1976d2'
  ctx.lineWidth = 2
  
  const barWidth = plotWidth / nSolutions
  const maxBarHeight = plotHeight * 0.9
  
  for (let i = 0; i < nSolutions; i++) {
    const x = margin.left + i * barWidth + barWidth / 2
    const barHeight = (distribution[i] / maxProb) * maxBarHeight
    const y = margin.top + plotHeight - barHeight
    
    // バーを描画
    ctx.fillRect(x - barWidth / 2 + 2, y, barWidth - 4, barHeight)
  }
  
  // P軸上にインデックスとw(P)の値を表示
  ctx.textAlign = 'center'
  ctx.textBaseline = 'top'
  for (let i = 0; i < nSolutions; i++) {
    const x = margin.left + i * barWidth + barWidth / 2
    const y = margin.top + plotHeight + 5
    const isMinW = i === minWIndex.value
    
    // w(P)が最小のPだけ太字にする
    if (isMinW) {
      ctx.fillStyle = '#000'
      ctx.font = 'bold 10px sans-serif'
    } else {
      ctx.fillStyle = '#000'
      ctx.font = '10px sans-serif'
    }
    
    // Pのインデックスとw(P)の値を表示
    ctx.fillText(`${i}`, x, y)
    ctx.fillText(`w=${wValues.value[i].toFixed(1)}`, x, y + 12)
  }
  
  // 軸ラベル
  ctx.fillStyle = '#000'
  ctx.font = '14px sans-serif'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText('P (解のインデックス)', margin.left + plotWidth / 2, margin.top + plotHeight + 45)
  
  ctx.save()
  ctx.translate(20, margin.top + plotHeight / 2)
  ctx.rotate(-Math.PI / 2)
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText('π(P)', 0, 0)
  ctx.restore()
  
  // タイトル
  ctx.fillStyle = '#000'
  ctx.font = '16px sans-serif'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'top'
  ctx.fillText(`ボルツマン分布 (T = ${temperature.value.toFixed(2)})`, margin.left + plotWidth / 2, 10)
  
  // T -> 0の場合の説明
  if (useDirac) {
    ctx.fillStyle = '#d32f2f'
    ctx.font = '12px sans-serif'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'top'
    ctx.fillText(`T → 0: Dirac測度 (最小w(P)の解のみ確率1)`, margin.left + plotWidth / 2, margin.top - 25)
  }
}

// 温度が変更されたときに再描画
watch(temperature, () => {
  draw()
})

// 初期化
initializeWValues()
watch(wValues, () => {
  draw()
}, { deep: true })

// マウント時に描画
import { onMounted } from 'vue'
onMounted(() => {
  draw()
})
</script>

<template>
  <div style="display: flex; flex-direction: column; align-items: center; gap: 1em;">
    <div style="display: flex; flex-direction: column; align-items: center; gap: 0.5em; width: 100%;">
      <label style="font-weight: bold; font-size: 14px;">
        温度 T: {{ temperature.toFixed(2) }}
      </label>
      <input
        v-model.number="temperature"
        type="range"
        min="0.01"
        max="50"
        step="0.01"
        style="width: 100%; max-width: 600px;"
      />
      <div style="display: flex; justify-content: space-between; width: 100%; max-width: 600px; font-size: 11px; color: #666;">
        <span>0.01</span>
        <span>25.0</span>
        <span>50.0</span>
      </div>
    </div>
    <canvas
      ref="canvasRef"
      :width="canvasWidth"
      :height="canvasHeight"
      style="border: 1px solid #ccc; border-radius: 8px;"
    ></canvas>
  </div>
</template>


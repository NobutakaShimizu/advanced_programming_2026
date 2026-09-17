<script setup>
import { onMounted, ref, watch } from 'vue'

const containerRef = ref(null)
const sliderRef = ref(null)
const sliderContainerRef = ref(null)
const varianceRef = ref(0.01)

let plotlyLoaded = false
let wheelHandler = null

// Plotly.jsを動的に読み込む
function loadPlotly() {
  return new Promise((resolve, reject) => {
    if (window.Plotly) {
      resolve()
      return
    }
    const script = document.createElement('script')
    script.src = 'https://cdn.plot.ly/plotly-2.26.0.min.js'
    script.onload = () => resolve()
    script.onerror = () => reject(new Error('Failed to load Plotly.js'))
    document.head.appendChild(script)
  })
}

// 計算量曲面を生成（複数のピークを持つ）
function generateComplexitySurface(variance) {
  const gridSize = 50
  const x = []
  const y = []
  const z = []
  
  // 入力空間を [0, 1] x [0, 1] として定義
  for (let i = 0; i <= gridSize; i++) {
    x.push(i / gridSize)
    y.push(i / gridSize)
  }
  
  // 複数のピークを定義（最悪ケースの入力に対応）
  const peaks = [
    { x: 0.2, y: 0.3, height: 10, width: 0.15 },
    { x: 0.7, y: 0.2, height: 12, width: 0.12 },
    { x: 0.5, y: 0.7, height: 15, width: 0.1 },
    { x: 0.8, y: 0.8, height: 11, width: 0.13 },
  ]
  
  const baseLevel = 3 // 平均時計算量レベル
  
  // 平滑化パラメータ（分散に応じて変化）
  const smoothingStrength = Math.min(1, variance * 15) // 0から1の間
  
  // z軸（計算量）を計算
  for (let i = 0; i <= gridSize; i++) {
    const row = []
    for (let j = 0; j <= gridSize; j++) {
      const xi = x[i]
      const yi = y[j]
      
      // 元の計算量（ピークを持つ）
      let complexity = baseLevel
      peaks.forEach(peak => {
        const dist = Math.sqrt((xi - peak.x) ** 2 + (yi - peak.y) ** 2)
        // 平滑化によってピークの高さと幅を調整
        const effectiveHeight = peak.height * (1 - smoothingStrength)
        const effectiveWidth = peak.width * (1 + smoothingStrength * 2)
        const contribution = effectiveHeight * Math.exp(-(dist ** 2) / (2 * effectiveWidth ** 2))
        complexity += contribution
      })
      
      // さらに、ガウシアン平滑化を適用（分散が大きいほど周囲の値と平均化）
      if (variance > 0.001) {
        const smoothingRadius = Math.sqrt(variance) * 8
        let smoothed = 0
        let weightSum = 0
        
        // 周囲の点をサンプリング（より細かく）
        const sampleRadius = Math.ceil(smoothingRadius)
        for (let di = -sampleRadius; di <= sampleRadius; di++) {
          for (let dj = -sampleRadius; dj <= sampleRadius; dj++) {
            const ni = Math.max(0, Math.min(gridSize, i + di))
            const nj = Math.max(0, Math.min(gridSize, j + dj))
            const nxi = x[ni]
            const nyi = y[nj]
            
            // 近傍点の計算量を計算
            let neighborComplexity = baseLevel
            peaks.forEach(peak => {
              const dist = Math.sqrt((nxi - peak.x) ** 2 + (nyi - peak.y) ** 2)
              const effectiveHeight = peak.height * (1 - smoothingStrength)
              const effectiveWidth = peak.width * (1 + smoothingStrength * 2)
              const contribution = effectiveHeight * Math.exp(-(dist ** 2) / (2 * effectiveWidth ** 2))
              neighborComplexity += contribution
            })
            
            // ガウシアン重み
            const dist2 = (di ** 2 + dj ** 2) / (gridSize ** 2)
            const weight = Math.exp(-dist2 / (2 * variance * 100))
            smoothed += neighborComplexity * weight
            weightSum += weight
          }
        }
        
        // 平滑化された値と元の値の重み付き平均
        const blendFactor = Math.min(1, variance * 25)
        complexity = complexity * (1 - blendFactor) + (smoothed / weightSum) * blendFactor
      }
      
      row.push(complexity)
    }
    z.push(row)
  }
  
  return { x, y, z }
}

// プロットを更新
function updatePlot() {
  if (!containerRef.value || !window.Plotly) return
  
  const variance = parseFloat(varianceRef.value)
  const { x, y, z } = generateComplexitySurface(variance)
  
  const data = [{
    x: x,
    y: y,
    z: z,
    type: 'surface',
    colorscale: 'Viridis',
    showscale: true,
    colorbar: {
      title: {
        text: '計算量',
        font: { size: 8 }
      },
      titleside: 'right',
      len: 0.7
    },
    hovertemplate: '入力: (%{x:.2f}, %{y:.2f})<br>計算量: %{z:.2f}<extra></extra>'
  }]
  
  const layout = {
    scene: {
      xaxis: { 
        title: '入力空間 (x)',
        range: [0, 1],
        titlefont: { size: 9 }
      },
      yaxis: { 
        title: '入力空間 (y)',
        range: [0, 1],
        titlefont: { size: 9 }
      },
      zaxis: { 
        title: '計算量',
        range: [0, 20],
        titlefont: { size: 9 }
      },
      camera: {
        eye: { x: 1.5, y: 1.5, z: 1.2 }
      },
      bgcolor: 'rgba(0,0,0,0)'
    },
    margin: { l: 0, r: 0, t: 0, b: 0 },
    height: 315,
    width: 420,
    paper_bgcolor: 'rgba(0,0,0,0)',
    plot_bgcolor: 'rgba(0,0,0,0)'
  }
  
  const config = {
    responsive: true,
    displayModeBar: false,
    scrollZoom: false // Plotlyのデフォルトのスクロールズームを無効化
  }
  
  Plotly.newPlot(containerRef.value, data, layout, config)
  
  // プロット要素にマウスホイールイベントを追加
  const plotElement = containerRef.value.querySelector('.plotly')
  if (plotElement) {
    // 既存のイベントリスナーを削除
    if (wheelHandler) {
      plotElement.removeEventListener('wheel', wheelHandler)
    }
    
    // 新しいイベントリスナーを追加
    wheelHandler = (e) => {
      e.preventDefault()
      e.stopPropagation()
      const delta = e.deltaY > 0 ? -0.0005 : 0.0005
      const newValue = Math.max(0, Math.min(0.08, parseFloat(varianceRef.value) + delta))
      varianceRef.value = parseFloat(newValue.toFixed(4))
    }
    plotElement.addEventListener('wheel', wheelHandler, { passive: false })
  }
}

onMounted(async () => {
  try {
    await loadPlotly()
    plotlyLoaded = true
    updatePlot()
  } catch (error) {
    console.error('Failed to load Plotly.js:', error)
  }
})

// スライダーがマウントされた後にイベントリスナーを設定
watch([sliderRef, sliderContainerRef], ([slider, container]) => {
  if (slider && container) {
    // スライダーコンテナ上でのマウスホイールイベントを無効化（ドラッグ操作を優先）
    const handleWheel = (e) => {
      // スライダーまたはその子要素上でのみイベントを停止
      if (e.target === slider || slider.contains(e.target)) {
        e.stopPropagation()
        e.preventDefault()
      }
    }
    container.addEventListener('wheel', handleWheel, { passive: false })
    
    // スライダー自体でもイベントを停止
    slider.addEventListener('wheel', (e) => {
      e.stopPropagation()
      e.preventDefault()
    }, { passive: false })
  }
}, { immediate: true })

// 分散が変更されたときにプロットを更新
watch(varianceRef, () => {
  if (plotlyLoaded) {
    updatePlot()
  }
})
</script>

<template>
  <div style="display: flex; flex-direction: column; align-items: center; gap: 0.4em;">
    <div ref="containerRef" style="width: 100%; cursor: ns-resize;"></div>
    <div ref="sliderContainerRef" style="width: 100%; padding: 0 1em;">
      <label style="display: block; margin-bottom: 0.3em; font-size: 0.85em; font-weight: 500;">
        ノイズの分散: <span style="font-weight: bold; color: #1976d2;">{{ parseFloat(varianceRef).toFixed(4) }}</span>
      </label>
      <input
        ref="sliderRef"
        v-model.number="varianceRef"
        type="range"
        min="0"
        max="0.08"
        step="0.0005"
        style="width: 100%; cursor: pointer;"
      />
      <div style="display: flex; justify-content: space-between; font-size: 0.7em; color: #666; margin-top: 0.15em;">
        <span>小さい (最悪時)</span>
        <span>大きい (平均時)</span>
      </div>
    </div>
  </div>
</template>


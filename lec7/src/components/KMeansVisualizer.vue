<script setup>
import { onMounted, ref } from 'vue'

const containerRef = ref(null)
const playBtnRef = ref(null)
const resetBtnRef = ref(null)
const clearCentersBtnRef = ref(null)

onMounted(() => {
  const container = containerRef.value
  const playBtn = playBtnRef.value
  const resetBtn = resetBtnRef.value
  const clearCentersBtn = clearCentersBtnRef.value
  if (!container || !playBtn || !resetBtn || !clearCentersBtn) return

  // 設定
  const width = 900
  const height = 600
  const padding = 40
  const pointRadius = 4
  const centerRadius = 8
  const numPoints = 200

  // 色のパレット（中心点とデータ点に使用）
  const colors = [
    '#e53e3e', // 赤
    '#3182ce', // 青
    '#38a169', // 緑
    '#d69e2e', // 黄
    '#805ad5', // 紫
  ]

  let svg = null
  let dataPoints = []
  let centers = []
  let assignments = []
  let step = 0
  let isPlaying = false
  let animationTimer = null
  let k = 0

  // データ点の生成（2〜5個のクラスタに分かれるように）
  function generateDataPoints() {
    // クラスタ数をランダムに決定（2〜5）
    k = 2 + Math.floor(Math.random() * 4)
    
    dataPoints = []
    const clusterCenters = []
    const clusterStd = 0.25 // クラスタの標準偏差（フレーム全体に散らばるように調整）
    const margin = 0.05 // フレーム端からのマージン
    
    // 各クラスタの中心を生成（フレーム全体に均一に配置）
    for (let i = 0; i < k; i++) {
      clusterCenters.push({
        x: margin + Math.random() * (1 - 2 * margin),
        y: margin + Math.random() * (1 - 2 * margin),
      })
    }
    
    // 各クラスタからデータ点を生成
    const pointsPerCluster = Math.floor(numPoints / k)
    const remainder = numPoints % k
    
    for (let i = 0; i < k; i++) {
      const numPointsInCluster = pointsPerCluster + (i < remainder ? 1 : 0)
      const center = clusterCenters[i]
      
      for (let j = 0; j < numPointsInCluster; j++) {
        // 正規分布に近いランダムな点を生成（フレーム内に収まるまでサンプリングし直す）
        let x, y
        let attempts = 0
        const maxAttempts = 100 // 無限ループを防ぐ
        
        do {
          const angle = Math.random() * 2 * Math.PI
          const radius = Math.random() * clusterStd
          x = center.x + radius * Math.cos(angle)
          y = center.y + radius * Math.sin(angle)
          attempts++
        } while ((x < margin || x > 1 - margin || y < margin || y > 1 - margin) && attempts < maxAttempts)
        
        // 最大試行回数に達した場合は、フレーム内に収まるように丸める（フォールバック）
        if (attempts >= maxAttempts) {
          x = Math.max(margin, Math.min(1 - margin, x))
          y = Math.max(margin, Math.min(1 - margin, y))
        }
        
        dataPoints.push({ x, y })
      }
    }
    
    // データ点を少しシャッフル
    for (let i = dataPoints.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [dataPoints[i], dataPoints[j]] = [dataPoints[j], dataPoints[i]]
    }
  }

  // 初期中心点の生成（空にする - ユーザーがクリックで追加）
  function initializeCenters() {
    centers = []
    step = 0
    assignments = []
  }

  // 座標変換関数（正規化座標→画面座標）
  const toX = (x) => padding + x * (width - 2 * padding)
  const toY = (y) => padding + y * (height - 2 * padding)

  // 座標変換関数（画面座標→正規化座標）
  const fromX = (x) => (x - padding) / (width - 2 * padding)
  const fromY = (y) => (y - padding) / (height - 2 * padding)

  // ユークリッド距離の二乗
  function distanceSquared(p1, p2) {
    const dx = p1.x - p2.x
    const dy = p1.y - p2.y
    return dx * dx + dy * dy
  }

  // 割り当てステップ：各データ点を最も近い中心点に割り当て
  function assignPoints() {
    assignments = dataPoints.map(point => {
      let minDist = Infinity
      let closestCenter = 0
      
      for (let j = 0; j < centers.length; j++) {
        const dist = distanceSquared(point, centers[j])
        if (dist < minDist) {
          minDist = dist
          closestCenter = j
        }
      }
      
      return closestCenter
    })
  }

  // 更新ステップ：各中心点を割り当てられたデータ点の平均に更新
  function updateCenters() {
    for (let j = 0; j < centers.length; j++) {
      const assignedPoints = dataPoints.filter((_, i) => assignments[i] === j)
      
      if (assignedPoints.length > 0) {
        const sumX = assignedPoints.reduce((sum, p) => sum + p.x, 0)
        const sumY = assignedPoints.reduce((sum, p) => sum + p.y, 0)
        centers[j].x = sumX / assignedPoints.length
        centers[j].y = sumY / assignedPoints.length
      }
    }
  }

  // 中心点を追加（クリック位置に最も近いデータ点を中心点とする）
  function addCenter(x, y) {
    if (dataPoints.length === 0) return
    
    // 正規化座標に変換
    const normalizedX = Math.max(0, Math.min(1, fromX(x)))
    const normalizedY = Math.max(0, Math.min(1, fromY(y)))
    
    // クリック位置に最も近いデータ点を見つける
    let minDist = Infinity
    let nearestPoint = null
    
    dataPoints.forEach(point => {
      const dist = distanceSquared(
        { x: normalizedX, y: normalizedY },
        point
      )
      if (dist < minDist) {
        minDist = dist
        nearestPoint = point
      }
    })
    
    // 最も近いデータ点の位置を中心点として追加
    if (nearestPoint) {
      centers.push({ x: nearestPoint.x, y: nearestPoint.y })
      step = 0
      
      // 中心点がある場合は割り当てを実行
      if (centers.length > 0) {
        assignPoints()
      }
      
      render()
    }
  }

  // 中心点を削除（クリック位置に最も近い中心点を削除）
  function removeNearestCenter(x, y) {
    if (centers.length === 0) return
    
    const normalizedX = fromX(x)
    const normalizedY = fromY(y)
    
    let minDist = Infinity
    let nearestIndex = -1
    
    centers.forEach((center, index) => {
      const dist = distanceSquared(
        { x: normalizedX, y: normalizedY },
        center
      )
      if (dist < minDist) {
        minDist = dist
        nearestIndex = index
      }
    })
    
    // 中心点がクリック位置から十分近い場合のみ削除（中心点の半径内）
    const clickRadius = 0.05 // 正規化座標での半径
    if (minDist < clickRadius * clickRadius) {
      centers.splice(nearestIndex, 1)
      step = 0
      
      if (centers.length > 0) {
        assignPoints()
      } else {
        assignments = []
      }
      
      render()
    }
  }

  // 中心点をクリア
  function clearCenters() {
    centers = []
    step = 0
    assignments = []
    render()
  }

  // 指定された座標に最も近い中心点のインデックスを返す
  function findNearestCenter(x, y) {
    if (centers.length === 0) return -1
    
    let minDist = Infinity
    let nearestIndex = 0
    
    centers.forEach((center, index) => {
      const dist = distanceSquared({ x, y }, center)
      if (dist < minDist) {
        minDist = dist
        nearestIndex = index
      }
    })
    
    return nearestIndex
  }

  // 描画
  function render() {
    // 既存のSVGを削除
    if (svg) {
      container.removeChild(svg)
    }

    // SVG要素の作成
    svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg')
    svg.setAttribute('width', width)
    svg.setAttribute('height', height)
    svg.style.border = '1px solid #ccc'
    svg.style.borderRadius = '8px'
    svg.style.backgroundColor = '#f9f9f9'
    svg.style.cursor = 'crosshair'

    // データ点の描画
    dataPoints.forEach((point, i) => {
      const color = assignments.length > 0 ? colors[assignments[i] % colors.length] : '#666'
      const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
      circle.setAttribute('cx', toX(point.x))
      circle.setAttribute('cy', toY(point.y))
      circle.setAttribute('r', pointRadius)
      circle.setAttribute('fill', color)
      circle.setAttribute('stroke', '#fff')
      circle.setAttribute('stroke-width', '1')
      svg.appendChild(circle)
    })

    // 中心点の描画
    centers.forEach((center, j) => {
      const color = colors[j % colors.length]
      const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
      circle.setAttribute('cx', toX(center.x))
      circle.setAttribute('cy', toY(center.y))
      circle.setAttribute('r', centerRadius)
      circle.setAttribute('fill', color)
      circle.setAttribute('stroke', '#fff')
      circle.setAttribute('stroke-width', '2')
      circle.style.pointerEvents = 'none'
      svg.appendChild(circle)
    })

    // SVGクリックイベント：中心点の追加/削除
    svg.addEventListener('click', (e) => {
      const rect = svg.getBoundingClientRect()
      // SVGは0.6倍にスケールされているので、元のサイズに戻す
      // SVGの実際のwidth/height属性を使用
      const svgWidth = parseFloat(svg.getAttribute('width'))
      const svgHeight = parseFloat(svg.getAttribute('height'))
      const x = (e.clientX - rect.left) * (svgWidth / rect.width)
      const y = (e.clientY - rect.top) * (svgHeight / rect.height)
      addCenter(x, y)
    })

    // SVG右クリックイベント：中心点の削除
    svg.addEventListener('contextmenu', (e) => {
      e.preventDefault()
      const rect = svg.getBoundingClientRect()
      // SVGは0.6倍にスケールされているので、元のサイズに戻す
      // SVGの実際のwidth/height属性を使用
      const svgWidth = parseFloat(svg.getAttribute('width'))
      const svgHeight = parseFloat(svg.getAttribute('height'))
      const x = (e.clientX - rect.left) * (svgWidth / rect.width)
      const y = (e.clientY - rect.top) * (svgHeight / rect.height)
      removeNearestCenter(x, y)
    })

    // ステップ情報の表示
    const text = document.createElementNS('http://www.w3.org/2000/svg', 'text')
    text.setAttribute('x', width - padding)
    text.setAttribute('y', padding)
    text.setAttribute('text-anchor', 'end')
    text.setAttribute('font-size', '14')
    text.setAttribute('fill', '#333')
    const phase = step === 0 ? '初期（割り当て済み）' : (step % 2 === 0 ? '更新後' : '割り当て後')
    text.textContent = `ステップ: ${step} (${phase})`
    svg.appendChild(text)

    container.appendChild(svg)
  }

  // 1ステップ進める
  function nextStep() {
    if (step % 2 === 0) {
      // 偶数ステップ：更新
      updateCenters()
      step++
    } else {
      // 奇数ステップ：割り当て
      assignPoints()
      step++
    }
    render()
  }

  // 再生/停止
  function togglePlay() {
    if (isPlaying) {
      // 停止
      if (animationTimer) {
        clearInterval(animationTimer)
        animationTimer = null
      }
      isPlaying = false
      playBtn.textContent = '再生'
    } else {
      // 再生
      animationTimer = setInterval(() => {
        nextStep()
      }, 1000)
      isPlaying = true
      playBtn.textContent = '停止'
    }
  }

  // リセット（データ点のみ再生成、中心点はクリア）
  function reset() {
    if (animationTimer) {
      clearInterval(animationTimer)
      animationTimer = null
    }
    isPlaying = false
    playBtn.textContent = '再生'
    generateDataPoints()
    clearCenters()
  }

  // ボタンのイベント
  playBtn.addEventListener('click', togglePlay)
  resetBtn.addEventListener('click', reset)
  clearCentersBtn.addEventListener('click', clearCenters)

  // 初期化
  generateDataPoints()
  initializeCenters()
  render()
})
</script>

<template>
  <div style="text-align: center;">
    <div style="transform: scale(0.6); transform-origin: top center; display: inline-block;">
      <div ref="containerRef" style="display: inline-block;"></div>
      <div style="text-align: center; margin-top: 0.5em; display: flex; gap: 0.5em; justify-content: center; flex-wrap: wrap;">
        <button 
          ref="playBtnRef"
          style="padding: 0.5em 1.5em; font-size: 0.9em; background-color: #1976d2; color: white; border: none; border-radius: 4px; cursor: pointer;"
        >
          再生
        </button>
        <button 
          ref="clearCentersBtnRef"
          style="padding: 0.5em 1.5em; font-size: 0.9em; background-color: #d69e2e; color: white; border: none; border-radius: 4px; cursor: pointer;"
        >
          中心点をクリア
        </button>
        <button 
          ref="resetBtnRef"
          style="padding: 0.5em 1.5em; font-size: 0.9em; background-color: #666; color: white; border: none; border-radius: 4px; cursor: pointer;"
        >
          リセット
        </button>
      </div>
      <div style="text-align: center; margin-top: 0.5em; font-size: 0.85em; color: #666;">
        左クリック: 中心点を追加 | 右クリック: 中心点を削除
      </div>
    </div>
  </div>
</template>


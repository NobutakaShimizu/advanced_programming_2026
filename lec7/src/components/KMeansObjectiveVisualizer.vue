<script setup>
import { onMounted, ref } from 'vue'

const containerRef = ref(null)
const resetBtnRef = ref(null)
const clearCentersBtnRef = ref(null)

onMounted(() => {
  const container = containerRef.value
  const resetBtn = resetBtnRef.value
  const clearCentersBtn = clearCentersBtnRef.value
  if (!container || !resetBtn || !clearCentersBtn) return

  // 設定（p12よりも小さめ）
  const width = 600
  const height = 400
  const padding = 40
  const pointRadius = 3
  const centerRadius = 7
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
  let k = 0
  let draggingCenterIndex = -1
  let isDragging = false

  // データ点の生成（一から作り直し）
  function generateDataPoints() {
    // クラスタ数をランダムに決定（2〜5）
    k = 2 + Math.floor(Math.random() * 4)
    
    dataPoints = []
    const margin = 0.05
    const clusterRadius = 0.25 // 各クラスタの半径（標準偏差を大きく）
    
    // 1. 2-5個のクラスタ点を選ぶ
    const clusterCenters = []
    const centerX = 0.5
    const centerY = 0.5
    const clusterSpread = 0.5 // クラスタ間の距離を大きくしてばらける
    const minDistance = 0.15 // クラスタ点間の最小距離（大きくして確実に分離）
    
    for (let i = 0; i < k; i++) {
      let clusterCenter
      let attempts = 0
      const maxAttempts = 500 // 試行回数を増やす
      
      // クラスタ点がフレーム内に収まり、他のクラスタ点から十分離れているまで選び直す
      do {
        clusterCenter = {
          x: centerX + (Math.random() - 0.5) * clusterSpread,
          y: centerY + (Math.random() - 0.5) * clusterSpread,
        }
        
        // フレーム内に収まるかチェック
        const inBounds = clusterCenter.x >= margin && clusterCenter.x <= 1 - margin &&
                        clusterCenter.y >= margin && clusterCenter.y <= 1 - margin
        
        // 他のクラスタ点から十分離れているかチェック
        let farEnough = true
        if (inBounds && clusterCenters.length > 0) {
          for (const existingCenter of clusterCenters) {
            const dist = Math.sqrt(
              Math.pow(clusterCenter.x - existingCenter.x, 2) +
              Math.pow(clusterCenter.y - existingCenter.y, 2)
            )
            if (dist < minDistance) {
              farEnough = false
              break
            }
          }
        }
        
        attempts++
        
        // 条件を満たしていればループを抜ける
        if (inBounds && farEnough) {
          break
        }
      } while (attempts < maxAttempts)
      
      // 最大試行回数に達した場合でも、フレーム内に収まるように調整して追加
      if (attempts >= maxAttempts) {
        clusterCenter.x = Math.max(margin, Math.min(1 - margin, clusterCenter.x))
        clusterCenter.y = Math.max(margin, Math.min(1 - margin, clusterCenter.y))
      }
      
      clusterCenters.push(clusterCenter)
    }
    
    // 2. 各クラスタ点の周囲を点で囲むように配置
    const pointsPerCluster = Math.floor(numPoints / k)
    const remainder = numPoints % k
    
    for (let i = 0; i < k; i++) {
      const numPointsInCluster = pointsPerCluster + (i < remainder ? 1 : 0)
      const center = clusterCenters[i]
      
      for (let j = 0; j < numPointsInCluster; j++) {
        let point
        let attempts = 0
        const maxAttempts = 100
        
        // 3. サンプリングした点がフレーム外になったら再度選び直す
        do {
          // クラスタ点の周囲にランダムに配置
          const angle = Math.random() * 2 * Math.PI
          const radius = Math.random() * clusterRadius
          point = {
            x: center.x + radius * Math.cos(angle),
            y: center.y + radius * Math.sin(angle),
          }
          attempts++
        } while (
          (point.x < margin || point.x > 1 - margin ||
           point.y < margin || point.y > 1 - margin) &&
          attempts < maxAttempts
        )
        
        // 最大試行回数に達した場合は、フレーム内に収まるように調整
        if (attempts >= maxAttempts) {
          point.x = Math.max(margin, Math.min(1 - margin, point.x))
          point.y = Math.max(margin, Math.min(1 - margin, point.y))
        }
        
        dataPoints.push(point)
      }
    }
    
    // データ点をシャッフル
    for (let i = dataPoints.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [dataPoints[i], dataPoints[j]] = [dataPoints[j], dataPoints[i]]
    }
  }

  // 初期中心点の生成（空にする - ユーザーがクリックで追加）
  function initializeCenters() {
    centers = []
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

  // 目的関数Φ(C)の値を計算
  function calculateObjective() {
    if (centers.length === 0 || assignments.length === 0) return 0
    
    let sum = 0
    dataPoints.forEach((point, i) => {
      const centerIndex = assignments[i]
      if (centerIndex < centers.length) {
        sum += distanceSquared(point, centers[centerIndex])
      }
    })
    
    return sum
  }

  // 中心点を追加（クリック位置に中心点を追加、データ点とは限らない）
  function addCenter(x, y) {
    // 正規化座標に変換
    const normalizedX = Math.max(0, Math.min(1, fromX(x)))
    const normalizedY = Math.max(0, Math.min(1, fromY(y)))
    
    // クリック位置に中心点を追加（データ点とは限らない）
    centers.push({ x: normalizedX, y: normalizedY })
    
    // 中心点がある場合は割り当てを実行
    if (centers.length > 0) {
      assignPoints()
    }
    
    render()
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
    
    // 中心点がクリック位置から十分近い場合のみ削除
    // 中心点の半径を考慮して判定範囲を広げる
    const clickRadius = 0.12 // 中心点の半径を考慮して大きくする
    if (minDist < clickRadius * clickRadius) {
      centers.splice(nearestIndex, 1)
      
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
    assignments = []
    render()
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
      circle.style.cursor = 'move'
      circle.style.pointerEvents = 'all'
      
      // 中心点のドラッグ開始
      circle.addEventListener('mousedown', (e) => {
        e.stopPropagation() // クリックイベントの伝播を防ぐ
        isDragging = true
        draggingCenterIndex = j
        
        const rect = svg.getBoundingClientRect()
        const svgWidth = parseFloat(svg.getAttribute('width'))
        const svgHeight = parseFloat(svg.getAttribute('height'))
        const startX = (e.clientX - rect.left) * (svgWidth / rect.width)
        const startY = (e.clientY - rect.top) * (svgHeight / rect.height)
        
        // マウス移動イベント
        const onMouseMove = (e) => {
          if (!isDragging || draggingCenterIndex !== j) return
          
          const rect = svg.getBoundingClientRect()
          const svgWidth = parseFloat(svg.getAttribute('width'))
          const svgHeight = parseFloat(svg.getAttribute('height'))
          const x = (e.clientX - rect.left) * (svgWidth / rect.width)
          const y = (e.clientY - rect.top) * (svgHeight / rect.height)
          
          // 正規化座標に変換
          const normalizedX = Math.max(0, Math.min(1, fromX(x)))
          const normalizedY = Math.max(0, Math.min(1, fromY(y)))
          
          // 中心点を移動
          centers[j].x = normalizedX
          centers[j].y = normalizedY
          
          // 割り当てを再計算
          assignPoints()
          render()
        }
        
        // マウスアップイベント
        const onMouseUp = () => {
          if (isDragging && draggingCenterIndex === j) {
            isDragging = false
            draggingCenterIndex = -1
            document.removeEventListener('mousemove', onMouseMove)
            document.removeEventListener('mouseup', onMouseUp)
          }
        }
        
        document.addEventListener('mousemove', onMouseMove)
        document.addEventListener('mouseup', onMouseUp)
      })
      
      svg.appendChild(circle)
    })

    // 目的関数の値を表示（左上）
    const objectiveValue = calculateObjective()
    const textBg = document.createElementNS('http://www.w3.org/2000/svg', 'rect')
    textBg.setAttribute('x', padding)
    textBg.setAttribute('y', padding - 20)
    textBg.setAttribute('width', 200)
    textBg.setAttribute('height', 30)
    textBg.setAttribute('fill', 'white')
    textBg.setAttribute('fill-opacity', '0.8')
    textBg.setAttribute('stroke', '#333')
    textBg.setAttribute('stroke-width', '1')
    svg.appendChild(textBg)

    const text = document.createElementNS('http://www.w3.org/2000/svg', 'text')
    text.setAttribute('x', padding + 10)
    text.setAttribute('y', padding)
    text.setAttribute('font-size', '16')
    text.setAttribute('fill', '#333')
    text.setAttribute('font-weight', 'bold')
    text.textContent = `Φ(C) = ${objectiveValue.toFixed(2)}`
    svg.appendChild(text)

    // SVGクリックイベント：中心点の追加（ドラッグ中でない場合のみ）
    svg.addEventListener('click', (e) => {
      // ドラッグ中はクリックイベントを無視
      if (isDragging) return
      
      const rect = svg.getBoundingClientRect()
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
      const svgWidth = parseFloat(svg.getAttribute('width'))
      const svgHeight = parseFloat(svg.getAttribute('height'))
      const x = (e.clientX - rect.left) * (svgWidth / rect.width)
      const y = (e.clientY - rect.top) * (svgHeight / rect.height)
      removeNearestCenter(x, y)
    })

    container.appendChild(svg)
  }

  // リセット（データ点のみ再生成、中心点はクリア）
  function reset() {
    generateDataPoints()
    clearCenters()
  }

  // ボタンのイベント
  resetBtn.addEventListener('click', reset)
  clearCentersBtn.addEventListener('click', clearCenters)

  // 初期化
  generateDataPoints()
  initializeCenters()
  render()
})
</script>

<template>
  <div style="display: flex; align-items: flex-start; gap: 1em; justify-content: center;">
    <div style="display: flex; flex-direction: column; gap: 0.5em;">
      <div style="font-size: 0.9em; margin-bottom: 0.5em;">
        左クリック: 中心点を追加<br>
        右クリック: 中心点を削除<br>
        ドラッグ: 中心点を移動
      </div>
      <button 
        ref="clearCentersBtnRef"
        style="padding: 0.5em 1.5em; font-size: 0.9em; background-color: #d69e2e; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
      >
        中心点をクリア
      </button>
      <button 
        ref="resetBtnRef"
        style="padding: 0.5em 1.5em; font-size: 0.9em; background-color: #666; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
      >
        リセット
      </button>
    </div>
    <div style="transform: scale(0.7); transform-origin: top left; display: inline-block;">
      <div ref="containerRef" style="display: inline-block;"></div>
    </div>
  </div>
</template>


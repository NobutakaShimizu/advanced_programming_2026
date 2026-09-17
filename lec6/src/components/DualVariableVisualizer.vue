<script setup>
import { onMounted, ref } from 'vue'

const containerRef = ref(null)
const stepBtnRef = ref(null)
const regenerateBtnRef = ref(null)

onMounted(() => {
  const container = containerRef.value
  const stepBtn = stepBtnRef.value
  const regenerateBtn = regenerateBtnRef.value
  if (!container || !stepBtn || !regenerateBtn) return

  // グラフの設定
  const gridSize = 4
  const nodeRadius = 20
  const cellWidth = 80
  const cellHeight = 80
  const startX = 50
  const startY = 50
  const width = startX * 2 + cellWidth * (gridSize - 1)
  const height = startY * 2 + cellHeight * (gridSize - 1)

  let svg = null
  let edges = [] // {from: {row, col}, to: {row, col}, weight: number}
  let potentials = [] // 2D配列で各頂点のポテンシャルを保持
  let selectedEdge = null // 選択された辺 {from: {row, col}, to: {row, col}}
  let sourceNode = { row: 0, col: 0 } // 始点

  // グリッドグラフの生成
  function generateGraph() {
    edges = []
    // 5x5グリッドの辺を生成（右方向と下方向）
    for (let row = 0; row < gridSize; row++) {
      for (let col = 0; col < gridSize; col++) {
        // 右方向の辺
        if (col < gridSize - 1) {
          edges.push({
            from: { row, col },
            to: { row, col: col + 1 },
            weight: Math.floor(Math.random() * 20) + 1
          })
        }
        // 下方向の辺
        if (row < gridSize - 1) {
          edges.push({
            from: { row, col },
            to: { row: row + 1, col },
            weight: Math.floor(Math.random() * 20) + 1
          })
        }
      }
    }
    
    // ポテンシャルの初期化
    initializePotentials()
    selectedEdge = null
  }

  // ポテンシャルの初期化
  function initializePotentials() {
    potentials = []
    for (let row = 0; row < gridSize; row++) {
      potentials[row] = []
      for (let col = 0; col < gridSize; col++) {
        if (row === sourceNode.row && col === sourceNode.col) {
          potentials[row][col] = 0
        } else {
          potentials[row][col] = Infinity
        }
      }
    }
  }

  // 頂点の座標を取得
  function getNodePosition(row, col) {
    return {
      x: startX + col * cellWidth,
      y: startY + row * cellHeight
    }
  }

  // ポテンシャルの表示用文字列
  function formatPotential(value) {
    if (value === Infinity) {
      return '∞'
    }
    return value.toString()
  }

  // グラフの描画
  function renderGraph() {
    if (svg) {
      container.removeChild(svg)
    }

    svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg')
    svg.setAttribute('width', width)
    svg.setAttribute('height', height)
    svg.style.border = '1px solid #ccc'
    svg.style.borderRadius = '8px'
    svg.style.backgroundColor = '#f9f9f9'

    // 選択された辺の背景（薄い色）
    if (selectedEdge) {
      const fromPos = getNodePosition(selectedEdge.from.row, selectedEdge.from.col)
      const toPos = getNodePosition(selectedEdge.to.row, selectedEdge.to.col)
      const bg = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      bg.setAttribute('x1', fromPos.x)
      bg.setAttribute('y1', fromPos.y)
      bg.setAttribute('x2', toPos.x)
      bg.setAttribute('y2', toPos.y)
      bg.setAttribute('stroke', '#BAE6FD')
      bg.setAttribute('stroke-width', '8')
      bg.style.pointerEvents = 'none'
      svg.appendChild(bg)
    }

    // 辺の描画
    edges.forEach((edge, index) => {
      const fromPos = getNodePosition(edge.from.row, edge.from.col)
      const toPos = getNodePosition(edge.to.row, edge.to.col)
      
      // クリック判定用の太い透明な線
      const hitArea = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      hitArea.setAttribute('x1', fromPos.x)
      hitArea.setAttribute('y1', fromPos.y)
      hitArea.setAttribute('x2', toPos.x)
      hitArea.setAttribute('y2', toPos.y)
      hitArea.setAttribute('stroke', 'transparent')
      hitArea.setAttribute('stroke-width', '15')
      hitArea.style.cursor = 'pointer'
      hitArea.dataset.edgeIndex = index

      // ポテンシャルを更新できるかチェック（無向グラフなので両方向をチェック）
      const y_u = potentials[edge.from.row][edge.from.col]
      const y_v = potentials[edge.to.row][edge.to.col]
      const w_e = edge.weight
      const canUpdateFromTo = y_v > y_u + w_e
      const canUpdateToFrom = y_u > y_v + w_e
      const canUpdate = canUpdateFromTo || canUpdateToFrom

      // 視覚的な線
      const line = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      line.setAttribute('x1', fromPos.x)
      line.setAttribute('y1', fromPos.y)
      line.setAttribute('x2', toPos.x)
      line.setAttribute('y2', toPos.y)
      line.setAttribute('stroke', canUpdate ? '#e53e3e' : '#333')
      line.setAttribute('stroke-width', canUpdate ? '3' : '2')
      line.style.pointerEvents = 'none'

      // 重みのテキスト
      const midX = (fromPos.x + toPos.x) / 2
      const midY = (fromPos.y + toPos.y) / 2
      // 縦の辺（下方向）の場合、ラベルを左にずらす
      const isVertical = edge.from.row < edge.to.row
      const labelX = isVertical ? midX - 15 : midX
      const text = document.createElementNS('http://www.w3.org/2000/svg', 'text')
      text.setAttribute('x', labelX)
      text.setAttribute('y', midY - 5)
      text.setAttribute('text-anchor', 'middle')
      text.setAttribute('font-size', '12')
      text.setAttribute('fill', '#333')
      text.textContent = edge.weight.toString()
      text.style.pointerEvents = 'none'

      // クリックイベント
      hitArea.addEventListener('click', () => {
        selectedEdge = {
          from: { ...edge.from },
          to: { ...edge.to }
        }
        renderGraph()
      })

      svg.appendChild(hitArea)
      svg.appendChild(line)
      svg.appendChild(text)
    })

    // 頂点の描画
    for (let row = 0; row < gridSize; row++) {
      for (let col = 0; col < gridSize; col++) {
        const pos = getNodePosition(row, col)
        const isSource = row === sourceNode.row && col === sourceNode.col
        
        // 円
        const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
        circle.setAttribute('cx', pos.x)
        circle.setAttribute('cy', pos.y)
        circle.setAttribute('r', nodeRadius)
        circle.setAttribute('fill', isSource ? '#4caf50' : '#1976d2')
        circle.setAttribute('stroke', '#fff')
        circle.setAttribute('stroke-width', '2')
        svg.appendChild(circle)

        // ポテンシャルのテキスト
        const text = document.createElementNS('http://www.w3.org/2000/svg', 'text')
        text.setAttribute('x', pos.x)
        text.setAttribute('y', pos.y + 5)
        text.setAttribute('text-anchor', 'middle')
        text.setAttribute('font-size', '14')
        text.setAttribute('font-weight', 'bold')
        text.setAttribute('fill', '#fff')
        text.textContent = formatPotential(potentials[row][col])
        svg.appendChild(text)
      }
    }

    container.appendChild(svg)
  }

  // 一コマ送りボタンのイベント
  stepBtn.addEventListener('click', () => {
    if (!selectedEdge) return

    const fromRow = selectedEdge.from.row
    const fromCol = selectedEdge.from.col
    const toRow = selectedEdge.to.row
    const toCol = selectedEdge.to.col

    // 選択された辺の重みを取得（無向グラフなので両方向をチェック）
    const edge = edges.find(e => 
      (e.from.row === fromRow && e.from.col === fromCol &&
       e.to.row === toRow && e.to.col === toCol) ||
      (e.from.row === toRow && e.from.col === toCol &&
       e.to.row === fromRow && e.to.col === fromCol)
    )

    if (!edge) return

    // ポテンシャルの更新（無向グラフなので両方向をチェック）
    const y_u = potentials[fromRow][fromCol]
    const y_v = potentials[toRow][toCol]
    const w_e = edge.weight

    // y_v > y_u + w(e) ならば y_v := y_u + w(e)
    if (y_v > y_u + w_e) {
      potentials[toRow][toCol] = y_u + w_e
    }
    // y_u > y_v + w(e) ならば y_u := y_v + w(e)
    else if (y_u > y_v + w_e) {
      potentials[fromRow][fromCol] = y_v + w_e
    }

    renderGraph()
  })

  // 再生成ボタンのイベント
  regenerateBtn.addEventListener('click', () => {
    generateGraph()
    renderGraph()
  })

  // 初期グラフの生成と描画
  generateGraph()
  renderGraph()
})
</script>

<template>
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; align-items: start;">
    <!-- 左のグリッド: グラフ -->
    <div style="display: flex; flex-direction: column; align-items: center;">
      <div ref="containerRef" style="display: inline-block;"></div>
    </div>
    
    <!-- 右のグリッド: ボタンと操作説明 -->
    <div style="display: flex; flex-direction: column; gap: 1em;">
      <div style="font-size: 0.9em; color: #333;">
        <h3 style="margin-top: 0; font-size: 1.1em; color: #333; margin-bottom: 0.5em;">操作方法</h3>
        <ul style="line-height: 1.8; padding-left: 1.2em; margin: 0;">
          <li>辺をクリック: 辺の選択</li>
          <li>「ポテンシャルの更新」: 選択した辺に沿ってポテンシャルを更新</li>
          <li>「再生成」: ポテンシャルを初期化し、グラフの重みを再生成</li>
          <li>赤い辺: y<sub>v</sub>の値を更新できる辺</li>
        </ul>
      </div>
      
      <!-- ボタン -->
      <div style="display: flex; flex-direction: column; gap: 0.8em; margin-top: 1em;">
        <button 
          ref="stepBtnRef"
          style="padding: 0.8em 1.5em; font-size: 0.9em; background-color: #1976d2; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
        >
          ポテンシャルの更新
        </button>
        <button 
          ref="regenerateBtnRef"
          style="padding: 0.8em 1.5em; font-size: 0.9em; background-color: #666; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
        >
          再生成
        </button>
      </div>
    </div>
  </div>
</template>


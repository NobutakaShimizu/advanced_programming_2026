<script setup>
import { onMounted, ref } from 'vue'

const containerRef = ref(null)
const regenerateBtnRef = ref(null)

onMounted(() => {
  const container = containerRef.value
  const regenerateBtn = regenerateBtnRef.value
  if (!container || !regenerateBtn) return

  // グラフの設定
  const leftNodes = 6
  const rightNodes = 6
  const width = 600
  const height = 400
  const leftX = 150
  const rightX = 450
  const nodeRadius = 15
  const nodeSpacing = height / (leftNodes + 1)

  let svg = null
  let edges = []
  let matchingEdges = new Set()
  let edgeElements = []

  // ランダムな二部グラフの生成
  function generateGraph() {
    edges = []
    for (let i = 0; i < leftNodes; i++) {
      // 各左頂点から2-4本の辺を接続（適度な密度）
      const numEdges = 2 + Math.floor(Math.random() * 3)
      const connected = new Set()
      for (let j = 0; j < numEdges; j++) {
        let target
        do {
          target = Math.floor(Math.random() * rightNodes)
        } while (connected.has(target))
        connected.add(target)
        edges.push({ from: i, to: target })
      }
    }
  }

  // 辺がマッチングに追加可能かチェック
  function canAddToMatching(edgeIndex) {
    const edge = edges[edgeIndex]
    for (const matchingEdgeIndex of matchingEdges) {
      const matchingEdge = edges[matchingEdgeIndex]
      // 共有点を持つかチェック
      if (edge.from === matchingEdge.from || edge.to === matchingEdge.to) {
        return false
      }
    }
    return true
  }

  // グラフの描画
  function renderGraph() {
    // 既存のSVGを削除
    if (svg) {
      container.removeChild(svg)
    }
    matchingEdges.clear()
    edgeElements = []

    // SVG要素の作成
    svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg')
    svg.setAttribute('width', width)
    svg.setAttribute('height', height)
    svg.style.border = '1px solid #ccc'
    svg.style.borderRadius = '8px'
    svg.style.backgroundColor = '#f9f9f9'

    // 辺の描画
    edgeElements = edges.map((edge, index) => {
      const y1 = nodeSpacing * (edge.from + 1)
      const y2 = nodeSpacing * (edge.to + 1)
      
      // クリック判定用の太い透明な線（当たり判定を大きくする）
      const hitArea = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      hitArea.setAttribute('x1', leftX)
      hitArea.setAttribute('y1', y1)
      hitArea.setAttribute('x2', rightX)
      hitArea.setAttribute('y2', y2)
      hitArea.setAttribute('stroke', 'transparent')
      hitArea.setAttribute('stroke-width', '20') // 当たり判定を大きくする
      hitArea.style.cursor = 'pointer'
      hitArea.dataset.edgeIndex = index

      // 視覚的な細い線
      const line = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      line.setAttribute('x1', leftX)
      line.setAttribute('y1', y1)
      line.setAttribute('x2', rightX)
      line.setAttribute('y2', y2)
      line.setAttribute('stroke', '#333')
      line.setAttribute('stroke-width', '2')
      line.style.pointerEvents = 'none' // クリックイベントを無効化（hitAreaで処理）
      line.dataset.edgeIndex = index

      // クリックイベント（トグル動作）- hitAreaに設定
      hitArea.addEventListener('click', (e) => {
        if (matchingEdges.has(index)) {
          // 既にマッチングに含まれている場合は削除（黒に戻す）
          matchingEdges.delete(index)
          line.setAttribute('stroke', '#333')
          line.setAttribute('stroke-width', '2')
        } else {
          // マッチングに含まれていない場合は追加（赤にする）
          if (canAddToMatching(index)) {
            matchingEdges.add(index)
            line.setAttribute('stroke', '#e53e3e')
            line.setAttribute('stroke-width', '3')
            // マッチング辺を上に表示するため、再描画
            svg.removeChild(hitArea)
            svg.removeChild(line)
            svg.appendChild(hitArea)
            svg.appendChild(line)
          }
        }
      })

      // hitAreaを先に追加（下に配置）、lineを後に追加（上に配置）
      svg.appendChild(hitArea)
      svg.appendChild(line)
      return line
    })

    // 左側の頂点の描画
    for (let i = 0; i < leftNodes; i++) {
      const y = nodeSpacing * (i + 1)
      const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
      circle.setAttribute('cx', leftX)
      circle.setAttribute('cy', y)
      circle.setAttribute('r', nodeRadius)
      circle.setAttribute('fill', '#1976d2')
      circle.setAttribute('stroke', '#fff')
      circle.setAttribute('stroke-width', '2')
      svg.appendChild(circle)
    }

    // 右側の頂点の描画
    for (let i = 0; i < rightNodes; i++) {
      const y = nodeSpacing * (i + 1)
      const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
      circle.setAttribute('cx', rightX)
      circle.setAttribute('cy', y)
      circle.setAttribute('r', nodeRadius)
      circle.setAttribute('fill', '#1976d2')
      circle.setAttribute('stroke', '#fff')
      circle.setAttribute('stroke-width', '2')
      svg.appendChild(circle)
    }

    container.appendChild(svg)
  }

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
  <div style="text-align: center;">
    <div style="transform: scale(0.6); transform-origin: top center; display: inline-block;">
      <div ref="containerRef" style="display: inline-block;"></div>
      <div style="text-align: center; margin-top: 0.5em;">
        <button 
          ref="regenerateBtnRef"
          style="padding: 0.5em 1.5em; font-size: 0.9em; background-color: #1976d2; color: white; border: none; border-radius: 4px; cursor: pointer;"
        >
          再生成
        </button>
      </div>
    </div>
  </div>
</template>


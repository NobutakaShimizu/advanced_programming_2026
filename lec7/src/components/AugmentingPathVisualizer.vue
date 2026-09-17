<script setup>
import { onMounted, ref } from 'vue'

const containerRef = ref(null)
const xorBtnRef = ref(null)
const regenerateBtnRef = ref(null)
const resetBtnRef = ref(null)

onMounted(() => {
  const container = containerRef.value
  const xorBtn = xorBtnRef.value
  const regenerateBtn = regenerateBtnRef.value
  const resetBtn = resetBtnRef.value
  if (!container || !xorBtn || !regenerateBtn || !resetBtn) return

  // グラフの設定
  const leftNodes = 5
  const rightNodes = 5
  const width = 600
  const height = 450
  const leftX = 100
  const rightX = 500
  const nodeRadius = 15
  const nodeSpacing = (height / (leftNodes + 1)) * 1.2

  let svg = null
  let edges = []
  let matching = new Set() // 現在のマッチング
  let selectedEdges = new Set() // 選択された辺
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
    
    // 小さめな初期マッチングを生成
    matching.clear()
    const usedLeft = new Set()
    const usedRight = new Set()
    const shuffledEdges = [...edges].sort(() => Math.random() - 0.5)
    
    for (const edge of shuffledEdges) {
      if (!usedLeft.has(edge.from) && !usedRight.has(edge.to)) {
        const edgeIndex = edges.findIndex(e => e.from === edge.from && e.to === edge.to)
        matching.add(edgeIndex)
        usedLeft.add(edge.from)
        usedRight.add(edge.to)
        // 小さめなマッチングなので、2-3本程度で止める
        if (matching.size >= 2 && Math.random() < 0.5) break
      }
    }
    
    selectedEdges.clear()
  }

  // 選択された辺が増加路かどうかを判定
  function isAugmentingPath(selectedEdgeSet, matching) {
    if (selectedEdgeSet.size === 0) return false
    
    // マッチングに属する頂点を取得
    const matchedLeft = new Set()
    const matchedRight = new Set()
    for (const edgeIdx of matching) {
      matchedLeft.add(edges[edgeIdx].from)
      matchedRight.add(edges[edgeIdx].to)
    }
    
    // 選択された辺からグラフを構築
    const selectedEdgesList = Array.from(selectedEdgeSet).map(idx => ({
      idx,
      from: edges[idx].from,
      to: edges[idx].to,
      isMatching: matching.has(idx)
    }))
    
    // 隣接リストを構築
    const adj = new Map()
    for (let i = 0; i < leftNodes + rightNodes; i++) {
      adj.set(i, [])
    }
    
    for (const edge of selectedEdgesList) {
      const leftNode = edge.from
      const rightNode = edge.to + leftNodes
      adj.get(leftNode).push({ node: rightNode, edgeIdx: edge.idx, isMatching: edge.isMatching })
      adj.get(rightNode).push({ node: leftNode, edgeIdx: edge.idx, isMatching: edge.isMatching })
    }
    
    // 各頂点の次数をチェック（パスの場合、端点は次数1、中間点は次数2）
    const degree = new Map()
    for (let i = 0; i < leftNodes + rightNodes; i++) {
      degree.set(i, 0)
    }
    for (const edge of selectedEdgesList) {
      degree.set(edge.from, degree.get(edge.from) + 1)
      degree.set(edge.to + leftNodes, degree.get(edge.to + leftNodes) + 1)
    }
    
    // パスであるかチェック（次数が2を超える頂点がない、かつ連結）
    let endpointCount = 0
    for (let i = 0; i < leftNodes + rightNodes; i++) {
      const d = degree.get(i)
      if (d > 2) return false // 分岐がある
      if (d === 1) endpointCount++
    }
    if (endpointCount !== 2) return false // パスは端点が2つ
    
    // マッチングに属さない左頂点から探索
    for (let start = 0; start < leftNodes; start++) {
      if (matchedLeft.has(start) || degree.get(start) === 0) continue
      
      // BFSでパスを探索
      const queue = [{ node: start, path: [start], edges: [] }]
      const visited = new Set([start])
      
      while (queue.length > 0) {
        const { node, path, edges: pathEdges } = queue.shift()
        
        // 右頂点に到達した場合
        if (node >= leftNodes) {
          const rightIdx = node - leftNodes
          // マッチングに属さない右頂点で、選択されたすべての辺を含むパスかチェック
          if (!matchedRight.has(rightIdx) && pathEdges.length === selectedEdgeSet.size) {
            // 交互路の条件をチェック（交互にマッチング/非マッチング）
            // 増加路は奇数長で、最初と最後の辺が非マッチング
            if (pathEdges.length > 0 && pathEdges.length % 2 === 1) {
              let isValid = true
              for (let i = 0; i < pathEdges.length; i++) {
                const edgeIdx = pathEdges[i]
                const isInMatching = matching.has(edgeIdx)
                // 交互路: 0-indexedで偶数番目は非マッチング、奇数番目はマッチング
                if (i % 2 === 0 && isInMatching) {
                  isValid = false
                  break
                }
                if (i % 2 === 1 && !isInMatching) {
                  isValid = false
                  break
                }
              }
              if (isValid) return true
            }
          }
        }
        
        // 隣接頂点を探索
        const neighbors = adj.get(node) || []
        for (const neighbor of neighbors) {
          if (!visited.has(neighbor.node)) {
            // 交互路の条件: 現在の辺と次の辺が交互にマッチング/非マッチング
            if (pathEdges.length > 0) {
              const lastEdgeIdx = pathEdges[pathEdges.length - 1]
              const lastIsMatching = matching.has(lastEdgeIdx)
              if (lastIsMatching === neighbor.isMatching) continue // 交互でない
            }
            
            visited.add(neighbor.node)
            queue.push({
              node: neighbor.node,
              path: [...path, neighbor.node],
              edges: [...pathEdges, neighbor.edgeIdx]
            })
          }
        }
      }
    }
    
    return false
  }

  // グラフの描画
  function renderGraph() {
    if (svg) {
      container.removeChild(svg)
    }
    edgeElements = []

    svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg')
    svg.setAttribute('width', width)
    svg.setAttribute('height', height)
    svg.style.border = '1px solid #ccc'
    svg.style.borderRadius = '8px'
    svg.style.backgroundColor = '#f9f9f9'

    // 選択された辺の背景（薄空色）
    for (const edgeIdx of selectedEdges) {
      const edge = edges[edgeIdx]
      const y1 = nodeSpacing * (edge.from + 0.5)
      const y2 = nodeSpacing * (edge.to + 0.5)
      const bg = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      bg.setAttribute('x1', leftX)
      bg.setAttribute('y1', y1)
      bg.setAttribute('x2', rightX)
      bg.setAttribute('y2', y2)
      bg.setAttribute('stroke', '#BAE6FD')
      bg.setAttribute('stroke-width', '25')
      bg.style.pointerEvents = 'none'
      svg.appendChild(bg)
    }

    // 辺の描画
    edgeElements = edges.map((edge, index) => {
      const y1 = nodeSpacing * (edge.from + 0.5)
      const y2 = nodeSpacing * (edge.to + 0.5)
      
      // クリック判定用の太い透明な線
      const hitArea = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      hitArea.setAttribute('x1', leftX)
      hitArea.setAttribute('y1', y1)
      hitArea.setAttribute('x2', rightX)
      hitArea.setAttribute('y2', y2)
      hitArea.setAttribute('stroke', 'transparent')
      hitArea.setAttribute('stroke-width', '20')
      hitArea.style.cursor = 'pointer'
      hitArea.dataset.edgeIndex = index

      // 視覚的な線
      const line = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      line.setAttribute('x1', leftX)
      line.setAttribute('y1', y1)
      line.setAttribute('x2', rightX)
      line.setAttribute('y2', y2)
      const isMatchingEdge = matching.has(index)
      const baseStrokeWidth = isMatchingEdge ? '3' : '2'
      if (isMatchingEdge) {
        line.setAttribute('stroke', '#e53e3e')
        line.setAttribute('stroke-width', baseStrokeWidth)
      } else {
        line.setAttribute('stroke', '#333')
        line.setAttribute('stroke-width', baseStrokeWidth)
      }
      line.style.pointerEvents = 'none'
      line.dataset.edgeIndex = index
      line.dataset.isMatching = isMatchingEdge.toString()

      // マウスホバーイベント
      hitArea.addEventListener('mouseenter', () => {
        const currentWidth = parseFloat(line.getAttribute('stroke-width'))
        line.setAttribute('stroke-width', (currentWidth + 2).toString())
      })

      hitArea.addEventListener('mouseleave', () => {
        // 現在のマッチング状態に基づいて基本の太さを取得
        const isCurrentlyMatching = matching.has(index)
        const currentBaseWidth = isCurrentlyMatching ? '3' : '2'
        line.setAttribute('stroke-width', currentBaseWidth)
        // 色も更新（マッチング状態が変わった可能性があるため）
        if (isCurrentlyMatching) {
          line.setAttribute('stroke', '#e53e3e')
        } else {
          line.setAttribute('stroke', '#333')
        }
      })

      // マウスダウンイベント（右クリック検出のため）
      hitArea.addEventListener('mousedown', (e) => {
        if (e.button === 2) {
          // 右クリックで選択解除
          e.preventDefault()
          selectedEdges.delete(index)
          renderGraph()
        }
      })

      // クリックイベント（通常クリック）
      hitArea.addEventListener('click', (e) => {
        if (e.button === 0 || e.detail > 0) {
          // 通常クリックで選択/解除
          if (selectedEdges.has(index)) {
            selectedEdges.delete(index)
          } else {
            selectedEdges.add(index)
          }
          renderGraph()
        }
      })

      // 右クリックのデフォルト動作を無効化
      hitArea.addEventListener('contextmenu', (e) => {
        e.preventDefault()
      })

      svg.appendChild(hitArea)
      svg.appendChild(line)
      return line
    })

    // 左側の頂点の描画
    for (let i = 0; i < leftNodes; i++) {
      const y = nodeSpacing * (i + 0.5)
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
      const y = nodeSpacing * (i + 0.5)
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

  // XORボタンのイベント
  xorBtn.addEventListener('click', () => {
    if (isAugmentingPath(selectedEdges, matching)) {
      // 増加路の場合、M XOR P を計算
      for (const edgeIdx of selectedEdges) {
        if (matching.has(edgeIdx)) {
          matching.delete(edgeIdx)
        } else {
          matching.add(edgeIdx)
        }
      }
      selectedEdges.clear()
      renderGraph()
    }
  })

  // 再生成ボタンのイベント
  regenerateBtn.addEventListener('click', () => {
    generateGraph()
    renderGraph()
  })

  // リセットボタンのイベント
  resetBtn.addEventListener('click', () => {
    selectedEdges.clear()
    renderGraph()
  })

  // 初期グラフの生成と描画
  generateGraph()
  renderGraph()
})
</script>

<template>
  <div style="transform: scale(0.8); transform-origin: top center; display: inline-block; width: 100%;">
    <div style="display: grid; grid-template-columns: 5fr 5fr; gap: 2em; align-items: start;">
      <!-- 左のグリッド: 操作説明 -->
      <div style="display: flex; flex-direction: column; gap: 1em;">
        <div style="font-size: 0.9em; color: #333;">
          <ul style="line-height: 1.8; padding-left: 1.2em; margin: 0;">
            <li>辺クリック: 辺の選択/非選択を切り替える </li>
            <li>「XORをとる」: 選択した辺が増加路の場合、マッチングを更新</li>
            <li>「リセット」: 全ての辺の選択を解除</li>
            <li>「再生成」: グラフとマッチングを再生成</li>
          </ul>
        </div>
        
        <!-- ボタン -->
        <div style="display: flex; flex-direction: column; gap: 0.8em; margin-top: 1em;">
          <button 
            ref="xorBtnRef"
            style="padding: 0.8em 1.5em; font-size: 0.9em; background-color: #1976d2; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
          >
            XORをとる
          </button>
          <button 
            ref="resetBtnRef"
            style="padding: 0.8em 1.5em; font-size: 0.9em; background-color: #f59e0b; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
          >
            リセット
          </button>
          <button 
            ref="regenerateBtnRef"
            style="padding: 0.8em 1.5em; font-size: 0.9em; background-color: #666; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
          >
            再生成
          </button>
        </div>
      </div>
      
      <!-- 右のグリッド: グラフ -->
      <div style="display: flex; flex-direction: column; align-items: flex-start;">
        <div ref="containerRef" style="display: inline-block; margin-left: 2em;"></div>
      </div>
    </div>
  </div>
</template>


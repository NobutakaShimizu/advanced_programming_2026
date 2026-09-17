<script setup>
import { onMounted, ref, computed, nextTick, watch } from 'vue'
import katex from 'katex'
import 'katex/dist/katex.css'

const containerRef = ref(null)
const regenerateBtnRef = ref(null)
const matrixCellsRef = ref({})
const determinantRef = ref(null)
const titleTRef = ref(null)
const titleDetRef = ref(null)

// グラフの状態
const leftNodes = ['1', '2', '3', '4']
const rightNodes = ['a', 'b', 'c', 'd']
const edges = ref([])

// グラフの設定
const width = 400
const height = 400
const leftX = 100
const rightX = 300
const nodeRadius = 18
const nodeSpacing = height / (leftNodes.length + 1)

// Tutte行列の計算
const tutteMatrix = computed(() => {
  const matrix = Array(4).fill(null).map(() => Array(4).fill(0))
  
  edges.value.forEach(edge => {
    const leftIdx = leftNodes.indexOf(edge.from)
    const rightIdx = rightNodes.indexOf(edge.to)
    if (leftIdx !== -1 && rightIdx !== -1) {
      matrix[leftIdx][rightIdx] = `x_{${edge.from},${edge.to}}`
    }
  })
  
  return matrix
})

// 行列式の計算（シンボリック）
const determinant = computed(() => {
  const matrix = tutteMatrix.value
  const n = 4
  
  // 4x4行列の行列式を計算
  // det = Σ_{σ∈S_4} sgn(σ) * Π_{i=1}^4 T_{i,σ(i)}
  
  // すべての置換を生成（簡略化のため、非ゼロ項のみを計算）
  const terms = []
  
  // 完全マッチングに対応する項を探す
  function findMatchings(usedLeft, usedRight, currentMatching) {
    if (currentMatching.length === 4) {
      // 完全マッチングが見つかった
      let term = ''
      let sign = 1
      const perm = []
      
      currentMatching.forEach(([leftIdx, rightIdx]) => {
        perm[leftIdx] = rightIdx
      })
      
      // 置換の符号を計算（転倒数）
      let inversions = 0
      for (let i = 0; i < 4; i++) {
        for (let j = i + 1; j < 4; j++) {
          if (perm[i] > perm[j]) inversions++
        }
      }
      sign = inversions % 2 === 0 ? 1 : -1
      
      // 項を構築
      const factors = []
      currentMatching.forEach(([leftIdx, rightIdx]) => {
        const varName = matrix[leftIdx][rightIdx]
        if (varName && varName !== '0') {
          factors.push(varName)
        }
      })
      
      if (factors.length === 4) {
        // 変数を直接結合（掛け算を表す）
        const termStr = factors.join('')
        terms.push({ term: termStr, sign })
      }
      return
    }
    
    const leftIdx = currentMatching.length
    for (let rightIdx = 0; rightIdx < 4; rightIdx++) {
      if (!usedRight[rightIdx] && matrix[leftIdx][rightIdx] && matrix[leftIdx][rightIdx] !== '0') {
        usedRight[rightIdx] = true
        currentMatching.push([leftIdx, rightIdx])
        findMatchings(usedLeft, usedRight, currentMatching)
        currentMatching.pop()
        usedRight[rightIdx] = false
      }
    }
  }
  
  findMatchings([], Array(4).fill(false), [])
  
  // 項を整理
  if (terms.length === 0) {
    return '0'
  }
  
  const formattedTerms = terms.map(({ term, sign }) => {
    return sign === 1 ? term : `-${term}`
  })
  
  return formattedTerms.join(' + ').replace(/^\+ /, '').replace(/ \+ -/g, ' - ')
})

// ランダムな二部グラフの生成
function generateGraph() {
  edges.value = []
  for (let i = 0; i < leftNodes.length; i++) {
    // 各左頂点から2-3本の辺を接続
    const numEdges = 2 + Math.floor(Math.random() * 2)
    const connected = new Set()
    for (let j = 0; j < numEdges; j++) {
      let target
      do {
        target = Math.floor(Math.random() * rightNodes.length)
      } while (connected.has(target))
      connected.add(target)
      edges.value.push({ from: leftNodes[i], to: rightNodes[target] })
    }
  }
}

// グラフの描画
function renderGraph() {
  const container = containerRef.value
  if (!container) return
  
  // 既存のSVGを削除
  const existingSvg = container.querySelector('svg')
  if (existingSvg) {
    container.removeChild(existingSvg)
  }

  // SVG要素の作成
  const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg')
  svg.setAttribute('width', width)
  svg.setAttribute('height', height)
  svg.style.border = '1px solid #ccc'
  svg.style.borderRadius = '8px'
  svg.style.backgroundColor = '#f9f9f9'

  // 辺の描画
  edges.value.forEach(edge => {
    const leftIdx = leftNodes.indexOf(edge.from)
    const rightIdx = rightNodes.indexOf(edge.to)
    const y1 = nodeSpacing * (leftIdx + 1)
    const y2 = nodeSpacing * (rightIdx + 1)
    
    const line = document.createElementNS('http://www.w3.org/2000/svg', 'line')
    line.setAttribute('x1', leftX)
    line.setAttribute('y1', y1)
    line.setAttribute('x2', rightX)
    line.setAttribute('y2', y2)
    line.setAttribute('stroke', '#333')
    line.setAttribute('stroke-width', '2')
    svg.appendChild(line)
  })

  // 左側の頂点の描画
  leftNodes.forEach((label, i) => {
    const y = nodeSpacing * (i + 1)
    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
    circle.setAttribute('cx', leftX)
    circle.setAttribute('cy', y)
    circle.setAttribute('r', nodeRadius)
    circle.setAttribute('fill', '#1976d2')
    circle.setAttribute('stroke', '#fff')
    circle.setAttribute('stroke-width', '2')
    svg.appendChild(circle)
    
    // ラベル
    const text = document.createElementNS('http://www.w3.org/2000/svg', 'text')
    text.setAttribute('x', leftX)
    text.setAttribute('y', y + 5)
    text.setAttribute('text-anchor', 'middle')
    text.setAttribute('fill', '#fff')
    text.setAttribute('font-size', '14')
    text.setAttribute('font-weight', 'bold')
    text.textContent = label
    svg.appendChild(text)
  })

  // 右側の頂点の描画
  rightNodes.forEach((label, i) => {
    const y = nodeSpacing * (i + 1)
    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
    circle.setAttribute('cx', rightX)
    circle.setAttribute('cy', y)
    circle.setAttribute('r', nodeRadius)
    circle.setAttribute('fill', '#1976d2')
    circle.setAttribute('stroke', '#fff')
    circle.setAttribute('stroke-width', '2')
    svg.appendChild(circle)
    
    // ラベル
    const text = document.createElementNS('http://www.w3.org/2000/svg', 'text')
    text.setAttribute('x', rightX)
    text.setAttribute('y', y + 5)
    text.setAttribute('text-anchor', 'middle')
    text.setAttribute('fill', '#fff')
    text.setAttribute('font-size', '14')
    text.setAttribute('font-weight', 'bold')
    text.textContent = label
    svg.appendChild(text)
  })

  container.appendChild(svg)
}

onMounted(() => {
  const regenerateBtn = regenerateBtnRef.value
  if (!regenerateBtn) return

  // 再生成ボタンのイベント
  regenerateBtn.addEventListener('click', () => {
    generateGraph()
    renderGraph()
    // refをクリア
    matrixCellsRef.value = {}
    nextTick(() => {
      renderMath()
    })
  })

  // 初期グラフの生成と描画
  generateGraph()
  renderGraph()
  
  // 数式のレンダリング
  nextTick(() => {
    renderMath()
  })
})

// 数式のレンダリング関数
function renderMath() {
  nextTick(() => {
    // タイトルの数式をレンダリング
    if (titleTRef.value) {
      try {
        katex.render('T', titleTRef.value, {
          throwOnError: false,
          displayMode: false
        })
      } catch (e) {
        console.error('KaTeX render error for T:', e)
      }
    }
    
    if (titleDetRef.value) {
      try {
        katex.render('\\det(T)', titleDetRef.value, {
          throwOnError: false,
          displayMode: false
        })
      } catch (e) {
        console.error('KaTeX render error for det(T):', e)
      }
    }
    
    // 行列の各セルの数式をレンダリング
    Object.keys(matrixCellsRef.value).forEach(key => {
      const cellRef = matrixCellsRef.value[key]
      if (cellRef && cellRef.dataset && cellRef.dataset.math) {
        const mathStr = cellRef.dataset.math
        try {
          // x_{1,a} のような形式をそのままKaTeXでレンダリング
          katex.render(mathStr, cellRef, {
            throwOnError: false,
            displayMode: false
          })
        } catch (e) {
          console.error('KaTeX render error:', e, mathStr)
          cellRef.textContent = mathStr
        }
      }
    })
    
    // 行列式の数式をレンダリング
    if (determinantRef.value) {
      const detStr = determinant.value
      if (detStr && detStr !== '0') {
        try {
          // LaTeX記法に変換（既に正しい形式なのでそのまま使用）
          const latexStr = detStr
          katex.render(`\\det(T) = ${latexStr}`, determinantRef.value, {
            throwOnError: false,
            displayMode: false
          })
        } catch (e) {
          console.error('KaTeX render error:', e, detStr)
          determinantRef.value.textContent = `det(T) = ${detStr}`
        }
      } else {
        try {
          katex.render('\\det(T) = 0', determinantRef.value, {
            throwOnError: false,
            displayMode: false
          })
        } catch (e) {
          determinantRef.value.textContent = '0'
        }
      }
    }
  })
}

// 行列式が変更されたときに数式を再レンダリング
watch([tutteMatrix, determinant], () => {
  renderMath()
})
</script>

<template>
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; align-items: start;">
    <!-- 左側: グラフの描画 -->
    <div style="text-align: center;">
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

    <!-- 右側: Tutte行列と行列式 -->
    <div style="font-size: 0.9em;">
      <h3 style="margin-top: 0; font-size: 1.1em; color: #333;">
        Tutte行列 <span ref="titleTRef"></span>
      </h3>
      <div style="margin: 1em 0; padding: 1em; background-color: #f5f5f5; border-radius: 4px;">
        <div style="display: grid; grid-template-columns: auto 1fr; gap: 0.5em; align-items: center;">
          <div style="text-align: right; color: #666;">行\列</div>
          <div style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 0.5em;">
            <div style="text-align: center; font-weight: bold; color: #666;">a</div>
            <div style="text-align: center; font-weight: bold; color: #666;">b</div>
            <div style="text-align: center; font-weight: bold; color: #666;">c</div>
            <div style="text-align: center; font-weight: bold; color: #666;">d</div>
          </div>
        </div>
        <div v-for="(row, i) in tutteMatrix" :key="i" style="display: grid; grid-template-columns: auto 1fr; gap: 0.5em; align-items: center; margin-top: 0.5em;">
          <div style="text-align: right; font-weight: bold; color: #666;">{{ leftNodes[i] }}</div>
          <div style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 0.5em;">
            <div 
              v-for="(cell, j) in row" 
              :key="`${i}-${j}`"
              style="text-align: center; padding: 0.3em; background-color: white; border-radius: 3px; min-height: 1.5em; display: flex; align-items: center; justify-content: center;"
            >
              <span v-if="cell === 0" style="color: #999;">0</span>
              <span 
                v-else 
                :ref="el => { if (el) matrixCellsRef[`${i}-${j}`] = el }"
                :data-math="cell"
                style="font-family: 'Times New Roman', serif;"
              ></span>
            </div>
          </div>
        </div>
      </div>

      <h3 style="margin-top: 1.5em; font-size: 1.1em; color: #333;">
        行列式 <span ref="titleDetRef"></span>
      </h3>
      <div style="margin: 1em 0; padding: 1em; background-color: #f5f5f5; border-radius: 4px; font-family: 'Times New Roman', serif;">
        <div ref="determinantRef" style="text-align: center;">
          $\det(T) = {{ determinant }}$
        </div>
      </div>
    </div>
  </div>
</template>


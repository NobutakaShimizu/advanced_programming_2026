<script setup>
import { ref, computed, onMounted } from 'vue'

const rows = ['人1', '人2', '人3', '人4', '人5']
const cols = ['仕事a', '仕事b', '仕事c', '仕事d', '仕事e']

// 各セルの値を保持（5x5の配列）
const cellValues = ref([])

// 選択されたセルを保持（マッチング）
// 形式: { row: 0, col: 0 } の配列
const selectedCells = ref([])

// ランダムな値を生成してテーブルを初期化
function generateRandomValues() {
  cellValues.value = []
  for (let i = 0; i < 5; i++) {
    const row = []
    for (let j = 0; j < 5; j++) {
      row.push(Math.floor(Math.random() * 20) + 1) // 1から20の間
    }
    cellValues.value.push(row)
  }
  selectedCells.value = []
}

// セルが選択可能かチェック（マッチング制約）
function canSelectCell(row, col) {
  // 既に選択されているセルと同じ行・列にないかチェック
  return !selectedCells.value.some(cell => cell.row === row || cell.col === col)
}

// セルが既に選択されているかチェック
function isSelected(row, col) {
  return selectedCells.value.some(cell => cell.row === row && cell.col === col)
}

// セルをクリックしたときの処理
function toggleCell(row, col) {
  const index = selectedCells.value.findIndex(cell => cell.row === row && cell.col === col)
  
  if (index !== -1) {
    // 既に選択されている場合は削除
    selectedCells.value.splice(index, 1)
  } else {
    // 選択されていない場合、マッチング制約を満たす場合のみ追加
    if (canSelectCell(row, col)) {
      selectedCells.value.push({ row, col })
    }
  }
}

// 選択されたセルの総和を計算
const totalSum = computed(() => {
  return selectedCells.value.reduce((sum, cell) => {
    return sum + cellValues.value[cell.row][cell.col]
  }, 0)
})

// コンポーネントマウント時に初期化
onMounted(() => {
  generateRandomValues()
})
</script>

<template>
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em; align-items: start;">
    <!-- 左側: テーブル -->
    <div style="display: flex; flex-direction: column; align-items: center;">
      <table style="border-collapse: collapse; font-size: 0.9em;">
        <thead>
          <tr>
            <th style="width: 60px; height: 40px; border: 1px solid #ddd; background-color: #f5f5f5;"></th>
            <th 
              v-for="col in cols" 
              :key="col"
              style="width: 60px; height: 40px; border: 1px solid #ddd; background-color: #f5f5f5; font-weight: bold; font-size: 0.9em; color: #666; text-align: center;"
            >
              {{ col }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in rows" :key="rowIndex">
            <td 
              style="width: 60px; height: 50px; border: 1px solid #ddd; background-color: #f5f5f5; font-weight: bold; font-size: 0.9em; color: #333; text-align: center; padding: 0 8px;"
            >
              {{ row }}
            </td>
            <td
              v-for="(col, colIndex) in cols"
              :key="colIndex"
              :style="{
                width: '60px',
                height: '50px',
                border: '1px solid #ddd',
                textAlign: 'center',
                cursor: canSelectCell(rowIndex, colIndex) || isSelected(rowIndex, colIndex) ? 'pointer' : 'not-allowed',
                backgroundColor: isSelected(rowIndex, colIndex) ? '#b3e5fc' : '#fff',
                transition: 'background-color 0.2s',
                fontWeight: 'bold',
                fontSize: '1em',
                color: isSelected(rowIndex, colIndex) ? '#1976d2' : '#333'
              }"
              @click="toggleCell(rowIndex, colIndex)"
              :title="isSelected(rowIndex, colIndex) ? 'クリックで選択解除' : (canSelectCell(rowIndex, colIndex) ? 'クリックで選択' : '同じ行・列に既に選択されたセルがあります')"
            >
              {{ cellValues[rowIndex]?.[colIndex] || '' }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- 右側: 操作説明、総和、再生成ボタン -->
    <div style="display: flex; flex-direction: column; gap: 1.5em;">
      <div style="font-size: 0.9em; color: #333;">
        <h3 style="margin-top: 0; font-size: 1.1em; color: #333;">操作方法</h3>
        <ul style="line-height: 1.8; padding-left: 1.2em; margin: 0;">
          <li>セルをクリック: マッチングに追加（薄水色になる）</li>
          <li>選択済みセルをクリック: マッチングから削除</li>
          <li>同じ行・列に既に選択されたセルがある場合は選択できない</li>
        </ul>
      </div>
      
      <div style="font-size: 1em; color: #333;">
        <strong>選択したセルの総和: </strong>
        <span style="color: #1976d2; font-size: 1.2em; font-weight: bold;">{{ totalSum }}</span>
      </div>
      
      <button 
        @click="generateRandomValues"
        style="padding: 0.8em 1.5em; font-size: 0.9em; background-color: #1976d2; color: white; border: none; border-radius: 4px; cursor: pointer; white-space: nowrap;"
      >
        再生成
      </button>
    </div>
  </div>
</template>


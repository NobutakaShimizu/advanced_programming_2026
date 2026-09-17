<template>
  <div class="slidev-layout neversink">
    <!-- タイトル部分 - neversinkのtop-titleスタイル -->
    <div class="top-title" :class="titleBgClass">
      <div class="title" :class="titleColorClass">
        <slot name="title" />
      </div>
    </div>

    <!-- 2カラム部分 - neversinkのtwo-colsスタイル -->
    <div class="two-cols" :style="gridStyle">
      <div class="col-left">
        <slot name="left" />
      </div>
      <div class="col-right">
        <slot name="right" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  color: {
    type: String,
    default: 'primary'
  },
  ratio: {
    type: String,
    default: '1:1'
  }
})

const titleColorClass = computed(() => {
  const colorMap = {
    'primary': 'text-primary-500',
    'secondary': 'text-secondary-500',
    'accent': 'text-accent-500',
    'success': 'text-green-500',
    'warning': 'text-yellow-500',
    'error': 'text-red-500',
    'info': 'text-blue-500',
    'amber': 'text-amber-500',
    'amber-light': 'text-amber-500'
  }
  return colorMap[props.color] || 'text-primary-500'
})

const titleBgClass = computed(() => {
  const bgColorMap = {
    'primary': 'bg-primary-100',
    'secondary': 'bg-secondary-100',
    'accent': 'bg-accent-100',
    'success': 'bg-green-100',
    'warning': 'bg-yellow-100',
    'error': 'bg-red-100',
    'info': 'bg-blue-100',
    'amber': 'bg-amber-100',
    'amber-light': 'bg-amber-100'
  }
  return bgColorMap[props.color] || 'bg-primary-100'
})

const gridStyle = computed(() => {
  const [left, right] = props.ratio.split(':').map(Number)
  const total = left + right
  const leftPercent = (left / total) * 100
  const rightPercent = (right / total) * 100
  
  return {
    gridTemplateColumns: `${leftPercent}% ${rightPercent}%`
  }
})
</script>

<style scoped>
.slidev-layout {
  @apply h-full flex flex-col;
  margin: 0;
  padding: 0;
}

.top-title {
  @apply flex flex-col items-start justify-start text-left w-full;
  flex: 0 0 auto;
  margin: 0 0 1.5rem 0;
  padding: 0.125rem 1.5rem 0 1.4rem;
}

/* タイトル部分のテキストサイズを大きく調整 */
.title :deep(h1) {
  @apply text-3xl font-bold mb-2.5;
}

.title :deep(p),
.title :deep(span),
.title :deep(div) {
  @apply text-3xl font-bold mb-2.5;
}


.two-cols {
  @apply flex-1 grid grid-cols-2 gap-8 px-8 pb-8;
}

.col-left {
  @apply flex flex-col justify-start;
}

.col-right {
  @apply flex flex-col justify-start;
}

/* neversinkテーマのスタイルを継承 */
:deep(.title h1) {
  @apply text-4xl font-bold mb-2;
}

:deep(.subtitle p) {
  @apply text-lg opacity-80;
}
</style>

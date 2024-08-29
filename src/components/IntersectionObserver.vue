<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

let observer: any = null
const num = generateArray(0, 20)
const isInUpdateStack = ref<Array<string>>([])

function callback(entries: IntersectionObserverEntry[]) {
  entries.forEach((entry) => {
    const boxNumber = (entry.target as HTMLElement).textContent?.match(/\d+/)?.[0] || ''
    if (entry.isIntersecting) {
      if (!isInUpdateStack.value.includes(boxNumber)) {
        isInUpdateStack.value.push(boxNumber)
      }
    } else {
      const index = isInUpdateStack.value.indexOf(boxNumber)
      if (index !== -1) {
        isInUpdateStack.value.splice(index, 1)
      }
    }
  })
}

function generateArray(start: number, end: number): number[] {
  const result: number[] = []
  for (let i = start; i <= end; i++) {
    result.push(i)
  }
  return result
}

onMounted(() => {
  const boxes = document.querySelectorAll('.box')
  observer = new IntersectionObserver(callback, { threshold: [0, 1] })
  boxes.forEach((box) => observer.observe(box))
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<template>
  <div>
    <div>目前在更新列中有：「{{ isInUpdateStack.join(',') }}」</div>
    <div class="container">
      <div class="boxs">
        <div class="box" v-for="item in num" :key="item">box#{{ item }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  height: 450px;
  overflow-y: auto;
}
.boxs {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-top: 400px;
}

.box {
  padding: 4px 0;
  width: 100px;
  text-align: center;
  border-radius: 4px;
  background-color: lightblue;
  display: inline-block;
  color: white;
}
</style>

<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

let observer: IntersectionObserver | null = null
const num = generateArray(0, 20)
const isInUpdateStack = ref<Array<string>>([])
let inactivityTimer: number | null = null // Timer to detect inactivity

function onInactivity() {
  // Function to call when isInUpdateStack is inactive for 300ms
  alert('No changes in isInUpdateStack for 300ms.')
  // Add your desired function logic here
}

function startInactivityTimer() {
  // Clear the existing inactivity timer if it exists
  if (inactivityTimer !== null) {
    clearTimeout(inactivityTimer)
  }

  // Set a new inactivity timer for 300ms
  inactivityTimer = setTimeout(() => {
    onInactivity()
  }, 300)
}

function updateStack(str: string, add: boolean) {
  // Update the stack and start/reset the inactivity timer
  if (add) {
    if (!isInUpdateStack.value.includes(str)) {
      isInUpdateStack.value.push(str)
      startInactivityTimer() // Start or reset timer on update
    }
  } else {
    const idx = isInUpdateStack.value.indexOf(str)
    if (idx !== -1) {
      isInUpdateStack.value.splice(idx, 1)
      startInactivityTimer() // Start or reset timer on update
    }
  }
}

function callback(entries: IntersectionObserverEntry[]) {
  // Handle each intersection entry and update the stack
  entries.forEach((entry) => {
    const str = (entry.target as HTMLElement).getAttribute('data-class-id') ?? ''
    if (entry.isIntersecting) {
      updateStack(str, true) // Add to stack
    } else {
      updateStack(str, false) // Remove from stack
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
  // Clear the inactivity timer when the component is unmounted
  if (inactivityTimer !== null) {
    clearTimeout(inactivityTimer)
  }
})
</script>

<template>
  <div>
    <div>目前在更新列中有：「{{ isInUpdateStack.join(',') }}」</div>
    <div class="container">
      <div class="boxs">
        <div class="box" :data-class-id="index" v-for="(item, index) in num" :key="index">
          box#{{ index }}
        </div>
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

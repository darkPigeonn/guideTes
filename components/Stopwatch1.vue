<template>
  <div class="stopwatch">
    <h1 style="margin-bottom: 5px;">Jumlah Call : {{ count }}</h1>
    <h1>{{ formattedTime }}</h1>
    <button @click="startStop">{{ running ? 'Stop' : 'Start' }}</button>
    <button @click="reset" :disabled="!time">Reset</button>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// Define props
const props = defineProps({
  duration: Number,
  call: Number
})

// Reference to audio file
const alertSound = new Audio('/assets/pindah.mp3')
const finishSound = new Audio('/assets/selesai.mp3')

const time = ref(0)
const running = ref(false)
const count = ref(0)
let timer = null
let alertInterval = null
let startTime = null
let lastUpdateTime = null
let nextAlertTime = null

const formattedTime = computed(() => {
  const minutes = Math.floor(time.value / 60000)
  const seconds = Math.floor((time.value % 60000) / 1000)
  const milliseconds = time.value % 1000
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
})

const startStop = (event) => {
  event.preventDefault()
  if (running.value) {
    cancelAnimationFrame(timer)
    clearInterval(alertInterval)
  } else {
    startTime = performance.now() - time.value
    lastUpdateTime = performance.now()
    timer = requestAnimationFrame(updateTime)

    if (props.call > 0) {
      nextAlertTime = startTime + props.call * 1000 * (count.value + 1)
      alertInterval = setInterval(() => {
        if (time.value < props.duration * 1000) {
          alertSound.play()
          count.value++
          nextAlertTime += props.call * 1000
        }
      }, props.call * 1000)
    }
  }
  running.value = !running.value
}

const updateTime = () => {
  const now = performance.now()
  time.value = now - startTime

  if (time.value < props.duration * 1000) {
    timer = requestAnimationFrame(updateTime)
  } else {
    cancelAnimationFrame(timer)
    clearInterval(alertInterval)
    finishSound.play()
    running.value = false
    time.value = props.duration * 1000
  }
}

const reset = () => {
  cancelAnimationFrame(timer)
  clearInterval(alertInterval)
  time.value = 0
  running.value = false
  count.value = 0
}

// Watch for changes in props
watch(() => props.duration, (newVal) => {
  if (newVal * 1000 < time.value) {
    time.value = newVal * 1000
  }
})
</script>

<style scoped>
.stopwatch {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
h1 {
  color: white;
}
button {
  margin: 5px;
}
</style>

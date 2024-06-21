<template>
  <div class="stopwatch">
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
let timer = null
let alertInterval = null

const formattedTime = computed(() => {
  const minutes = Math.floor(time.value / 60000)
  const seconds = Math.floor((time.value % 60000) / 1000)
  const milliseconds = time.value % 1000
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}:${String(milliseconds).padStart(3, '0')}`
})

const startStop = (event) => {
  event.preventDefault();
  if (running.value) {
    clearInterval(timer)
    clearInterval(alertInterval)
  } else {
    timer = setInterval(() => {
      if (time.value < props.duration * 1000) {
        time.value += 10
      } else {
        clearInterval(timer)
        clearInterval(alertInterval)
        finishSound.play()
        running.value = false
      }
    }, 10)

    if (props.call > 0) {
      alertInterval = setInterval(() => {
        if (time.value < props.duration * 1000) {
          alertSound.play()
        }
      }, props.call * 1000)
    }
  }
  running.value = !running.value
}

const reset = () => {
  clearInterval(timer)
  time.value = 0
  running.value = false
}

// Watch for changes in props
watch(() => props.duration, (newVal) => {
  if (newVal * 1000 < time.value) {
    time.value = newVal * 1000
  }
})

watch(() => props.call, (newVal) => {
  // Logic for 'call' prop can be implemented here
  console.log(`Call prop changed: ${newVal}`)
})
</script>

<style scoped>
.stopwatch {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
h1{
  color:white
}
button {
  margin: 5px;
}
</style>

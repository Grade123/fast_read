<script setup lang="ts">
import { computed, ref, watch } from 'vue'

/**
 * Reactive message variable
 */
const message = ref('')
const words = computed(() => {
  if (message.value.trim() === '') {
    return []
  }
  return message.value.replace(/[\r\n]+/gm, ' ').trim().split(' ').filter(word => word !== '')
})

const wordIndex = ref(1)
const currentWord = computed(() => {
  if (words.value.length === 0) {
    return 'No Word'
  }
  return words.value[wordIndex.value - 1]
})

const speed = ref(1)
const running = ref(false)
const startStopText = computed(() => (running.value ? 'Stop' : 'Start'))


const middleCharInfo = computed(() => {
  const word = currentWord.value
  if (word === undefined) {
    return { before: '', middle: '', after: '' }
  }

  if ( word === 'No Word' || word.length === 0) {
    return { before: '', middle: '', after: '' }
  }
  const length = word.length
  const middleIndex = Math.floor((length) / 2 + (length % 2 === 0 ? -1 : 0))
  return {
    before: word.substring(0, middleIndex),
    middle: word[middleIndex],
    after: word.substring(middleIndex + 1)
  }
})

function onMoveToNextWord() {
  if (wordIndex.value < words.value.length) {
    wordIndex.value++
  }
}

function onMoveToPreviousWord() {
  if (wordIndex.value > 1) {
    wordIndex.value--
  }
}

function onResetWordIndex() {
  wordIndex.value = 1
}


function toggleRunning() {
  running.value = !running.value
}

let interval : number | null = null
const wpm = computed(() => (60 / speed.value) * 1000)

function stopInterval() {
  if (interval === undefined || interval === null) {
    return
  }

  clearInterval(interval)
  interval = null
}

function startInterval() {
  interval = setInterval(() => {
    if (running.value && wordIndex.value < words.value.length) {
      wordIndex.value++
    } else if (running.value) {
      running.value = false
    }
  }, wpm.value)
}

watch(running, (isRunning) => {
  if (isRunning) {
    startInterval()
  } else {
    stopInterval()
  }
})</script>

<template>
  <div id="container">
    <h1 :style="{ transform: currentWord!.length % 2 === 0 ? 'translateX(0.25em)' : 'translateX(0em)' }">{{ middleCharInfo.before }}<strong style="color: red;">{{ middleCharInfo.middle }}</strong>{{ middleCharInfo.after }}</h1>
    <textarea v-model="message" cols="70" rows="20">
    </textarea>
    <div id="buttonContainer">
      <button @click="onResetWordIndex">Reset</button>
      <button @click="onMoveToPreviousWord">Previous Word</button>
      <button @click="onMoveToNextWord">Next Word</button>
    </div>
    <input type="number" id="speed" v-model.number="speed"/>
    <input type="button" id="running" @click="toggleRunning" v-model="startStopText"/>
  </div>
</template>

<style scoped></style>

<style>
body {
  background-color:#18181b;
  margin: 0;
}

#app {
  width: 100vw;
  height: 100vh;

  display: flex;
  justify-content: center;
  align-items: center;

  color: #e5e5e5;
}

input, button, textarea {
  border-color: #404040;
  background-color: #18181b;
  font-size: 16px;
  color: #e5e5e5;
  padding: 1rem;
  border-style: solid;
  border-radius: 1rem;
}

#container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

#buttonContainer {
  display: flex;
  gap: 1rem;
}

h1 {
  color: white;
  font-size: 4rem;
  font-weight: normal;
  font-family: monospace;
}
</style>

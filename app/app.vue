<template>
  <div class="game-container">
    <header class="header">
      <h1 class="title">Pick Up Sticks</h1>
      <p class="subtitle">Select the sticks in the order they can be picked up</p>
    </header>

    <div v-if="!gameStarted" class="start-screen">
      <button @click="startGame" class="btn btn-primary">Start Game</button>
    </div>

    <div v-else-if="!gameComplete" class="game-area">
      <div class="info-bar">
        <div class="selected-count">Selected: {{ selectedSticks.length }}/{{ sticks.length }}</div>
        <button @click="resetGame" class="btn btn-small">Reset</button>
      </div>

      <svg
        ref="gameCanvas"
        class="canvas"
        :viewBox="`0 0 ${canvasWidth} ${canvasHeight}`"
        @click="handleCanvasClick"
      >
        <g v-for="stick in sticks" :key="stick.id">
          <line
            :x1="stick.x1"
            :y1="stick.y1"
            :x2="stick.x2"
            :y2="stick.y2"
            stroke="#000000"
            :stroke-width="stickWidth + 2"
            :stroke-linecap="'round'"
            class="stick-border"
          />
          <line
            :x1="stick.x1"
            :y1="stick.y1"
            :x2="stick.x2"
            :y2="stick.y2"
            :stroke="getStickColour(stick)"
            :stroke-width="stickWidth"
            :stroke-linecap="'round'"
            :class="{
              'stick': true,
              'selected': stick.selected,
              'selectable': !stick.selected && selectedSticks.length < sticks.length
            }"
            :data-stick-id="stick.id"
          />
          <text
            v-if="stick.selected"
            :x="(stick.x1 + stick.x2) / 2"
            :y="(stick.y1 + stick.y2) / 2"
            class="stick-number"
            text-anchor="middle"
            dominant-baseline="middle"
          >
            {{ stick.order }}
          </text>
        </g>
      </svg>

      <button
        v-if="selectedSticks.length === sticks.length"
        @click="checkOrder"
        class="btn btn-primary check-btn"
      >
        Check Order
      </button>
    </div>

    <div v-else class="results-screen">
      <div class="result-icon">{{ isCorrect ? '🎉' : '❌' }}</div>
      <h2 class="result-title">{{ isCorrect ? 'Correct!' : 'Try Again' }}</h2>
      <p class="result-message">
        {{ isCorrect
          ? 'You selected all the sticks in the correct order!'
          : 'The order wasn\'t quite right. Have another go!'
        }}
      </p>

      <div class="time-display">
        <p class="time-text">⏱️ Time: {{ formatTime(elapsedTime) }}</p>
      </div>

      <div class="result-details" v-if="!isCorrect">
        <p><strong>{{ incorrectCount }}</strong> {{ incorrectCount === 1 ? 'stick was' : 'sticks were' }} incorrect</p>
        <p>{{ incorrectCount === 1 ? 'This stick was' : 'These sticks were' }} still blocked by {{ incorrectCount === 1 ? 'another' : 'other' }} {{ incorrectCount === 1 ? 'stick' : 'sticks' }} on top when you tried to pick {{ incorrectCount === 1 ? 'it' : 'them' }} up.</p>
      </div>

      <div v-if="!isCorrect" class="results-canvas-container">
        <svg
          class="results-canvas"
          :viewBox="`0 0 ${canvasWidth} ${canvasHeight}`"
        >
          <g v-for="stick in sticks" :key="stick.id">
            <line
              :x1="stick.x1"
              :y1="stick.y1"
              :x2="stick.x2"
              :y2="stick.y2"
              stroke="#000000"
              :stroke-width="stickWidth + 2"
              :stroke-linecap="'round'"
              class="result-stick-border"
            />
            <line
              :x1="stick.x1"
              :y1="stick.y1"
              :x2="stick.x2"
              :y2="stick.y2"
              :stroke="getResultStickColour(stick)"
              :stroke-width="stickWidth"
              :stroke-linecap="'round'"
              class="result-stick"
            />
          </g>
        </svg>
      </div>

      <div class="button-group">
        <button @click="startGame" class="btn btn-primary">New Game</button>
        <button @click="shareResults" class="btn btn-secondary">Share Results</button>
      </div>

      <p v-if="shareMessage" class="share-message">{{ shareMessage }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const canvasWidth = 350
const canvasHeight = 500
const stickWidth = 8
const stickLength = 200
const numSticks = 10

const gameStarted = ref(false)
const gameComplete = ref(false)
const isCorrect = ref(false)
const sticks = ref([])
const selectedSticks = ref([])
const shareMessage = ref('')
const gameCanvas = ref(null)
const startTime = ref(null)
const elapsedTime = ref(0)
const incorrectCount = ref(0)
const incorrectStickIds = ref([])

const stickColours = [
  '#E74C3C', // red
  '#3498DB', // blue
  '#2ECC71', // green
  '#F39C12', // orange
  '#9B59B6', // purple
  '#1ABC9C', // turquoise
]

function generateSticks() {
  const maxAttempts = 100
  let attempts = 0

  while (attempts < maxAttempts) {
    const newSticks = []
    const centerRegionX = canvasWidth / 2
    const centerRegionY = canvasHeight / 2
    const regionRadius = Math.min(canvasWidth, canvasHeight) / 3

    for (let i = 0; i < numSticks; i++) {
      const angle = Math.random() * Math.PI * 2
      const distanceFromCenter = Math.random() * regionRadius
      const offsetAngle = Math.random() * Math.PI * 2

      const centerX = centerRegionX + Math.cos(offsetAngle) * distanceFromCenter
      const centerY = centerRegionY + Math.sin(offsetAngle) * distanceFromCenter

      const halfLength = stickLength / 2
      const x1 = centerX + Math.cos(angle) * halfLength
      const y1 = centerY + Math.sin(angle) * halfLength
      const x2 = centerX - Math.cos(angle) * halfLength
      const y2 = centerY - Math.sin(angle) * halfLength

      newSticks.push({
        id: i,
        x1,
        y1,
        x2,
        y2,
        angle,
        centerX,
        centerY,
        colour: stickColours[i % stickColours.length],
        selected: false,
        order: null,
        zIndex: i
      })
    }

    if (validateStickConfiguration(newSticks)) {
      return newSticks
    }

    attempts++
  }

  return generateSticks()
}

function validateStickConfiguration(sticks) {
  for (let i = 0; i < sticks.length; i++) {
    let hasOverlap = false
    for (let j = 0; j < sticks.length; j++) {
      if (i !== j && doSegmentsIntersect(sticks[i], sticks[j])) {
        hasOverlap = true
        break
      }
    }
    if (!hasOverlap) {
      return false
    }
  }

  return true
}

function doSegmentsIntersect(stick1, stick2) {
  const x1 = stick1.x1, y1 = stick1.y1, x2 = stick1.x2, y2 = stick1.y2
  const x3 = stick2.x1, y3 = stick2.y1, x4 = stick2.x2, y4 = stick2.y2

  const denom = (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4)
  if (Math.abs(denom) < 0.0001) {
    return false
  }

  const t = ((x1 - x3) * (y3 - y4) - (y1 - y3) * (x3 - x4)) / denom
  const u = -((x1 - x2) * (y1 - y3) - (y1 - y2) * (x1 - x3)) / denom

  const buffer = 0.1
  return t >= -buffer && t <= 1 + buffer && u >= -buffer && u <= 1 + buffer
}

function startGame() {
  gameStarted.value = true
  gameComplete.value = false
  isCorrect.value = false
  selectedSticks.value = []
  shareMessage.value = ''
  elapsedTime.value = 0
  incorrectCount.value = 0
  incorrectStickIds.value = []
  sticks.value = generateSticks()
  startTime.value = Date.now()
}

function resetGame() {
  selectedSticks.value = []
  sticks.value.forEach(stick => {
    stick.selected = false
    stick.order = null
  })
}

function handleCanvasClick(event) {
  if (selectedSticks.value.length >= sticks.value.length) return

  const target = event.target
  if (target.tagName !== 'line') return

  const stickId = parseInt(target.dataset.stickId)
  const stick = sticks.value.find(s => s.id === stickId)

  if (stick && !stick.selected) {
    stick.selected = true
    stick.order = selectedSticks.value.length + 1
    selectedSticks.value.push(stick)
  }
}

function checkOrder() {
  const validation = validateUserOrder(selectedSticks.value)
  isCorrect.value = validation.isValid
  incorrectCount.value = validation.incorrectCount
  incorrectStickIds.value = validation.incorrectStickIds

  if (startTime.value) {
    elapsedTime.value = Math.floor((Date.now() - startTime.value) / 1000)
  }

  gameComplete.value = true
}

function validateUserOrder(selectedOrder) {
  const remaining = [...sticks.value]
  let incorrectSticks = 0
  const incorrectIds = []

  for (let i = 0; i < selectedOrder.length; i++) {
    const pickedStick = selectedOrder[i]
    const pickedIndex = remaining.findIndex(s => s.id === pickedStick.id)

    if (pickedIndex === -1) {
      incorrectSticks++
      incorrectIds.push(pickedStick.id)
      continue
    }

    const isBlocked = remaining.some((other, j) => {
      if (j === pickedIndex) return false
      return other.zIndex > pickedStick.zIndex && doSegmentsIntersect(pickedStick, other)
    })

    if (isBlocked) {
      incorrectSticks++
      incorrectIds.push(pickedStick.id)
    } else {
      remaining.splice(pickedIndex, 1)
    }
  }

  return {
    isValid: incorrectSticks === 0,
    incorrectCount: incorrectSticks,
    incorrectStickIds: incorrectIds
  }
}

function formatTime(seconds) {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return mins > 0 ? `${mins}m ${secs}s` : `${secs}s`
}

function getStickColour(stick) {
  if (stick.selected) {
    return '#95A5A6'
  }
  return stick.colour
}

function getResultStickColour(stick) {
  if (incorrectStickIds.value.includes(stick.id)) {
    return stick.colour
  }
  return '#D3D3D3'
}

async function shareResults() {
  const timeText = `Time: ${formatTime(elapsedTime.value)}`
  const text = isCorrect.value
    ? `🎉 I completed Pick Up Sticks in the correct order in ${formatTime(elapsedTime.value)}! Can you beat it?\n\n${window.location.href}`
    : `I played Pick Up Sticks in ${formatTime(elapsedTime.value)} with ${incorrectCount.value} incorrect ${incorrectCount.value === 1 ? 'stick' : 'sticks'}! Can you do better?\n\n${window.location.href}`

  try {
    await navigator.clipboard.writeText(text)
    shareMessage.value = '✓ Copied to clipboard!'
    setTimeout(() => shareMessage.value = '', 3000)
  } catch (err) {
    shareMessage.value = 'Failed to copy to clipboard'
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

.game-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 2rem;
}

.title {
  font-size: 2rem;
  font-weight: 700;
  margin: 0 0 0.5rem 0;
}

.subtitle {
  font-size: 1rem;
  margin: 0;
  opacity: 0.9;
}

.start-screen,
.results-screen {
  background: white;
  border-radius: 1rem;
  padding: 2rem;
  text-align: center;
  max-width: 400px;
  width: 100%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.game-area {
  background: white;
  border-radius: 1rem;
  padding: 1rem;
  max-width: 400px;
  width: 100%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.info-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  font-weight: 600;
  color: #333;
}

.canvas {
  width: 100%;
  height: auto;
  background: #f8f9fa;
  border-radius: 0.5rem;
  display: block;
  touch-action: none;
}

.stick {
  cursor: pointer;
  transition: all 0.2s ease;
}

.stick.selected {
  opacity: 0.7;
}

.stick.selectable:hover {
  filter: brightness(1.2);
  stroke-width: 10;
}

.stick.selectable:active {
  filter: brightness(0.9);
}

.stick-number {
  font-size: 20px;
  font-weight: bold;
  fill: white;
  pointer-events: none;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.btn {
  background: #667eea;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  touch-action: manipulation;
}

.btn:hover {
  background: #5568d3;
  transform: translateY(-1px);
}

.btn:active {
  transform: translateY(0);
}

.btn-small {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.btn-primary {
  background: #667eea;
}

.btn-secondary {
  background: #764ba2;
}

.check-btn {
  width: 100%;
  margin-top: 1rem;
}

.button-group {
  display: flex;
  gap: 0.5rem;
  flex-direction: column;
}

.result-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.result-title {
  font-size: 1.75rem;
  color: #333;
  margin: 0 0 1rem 0;
}

.result-message {
  color: #666;
  margin: 0 0 1rem 0;
  line-height: 1.5;
}

.time-display {
  margin-bottom: 1.5rem;
}

.time-text {
  font-size: 1.25rem;
  font-weight: 600;
  color: #667eea;
  margin: 0;
}

.result-details {
  background: #f8f9fa;
  padding: 1rem;
  border-radius: 0.5rem;
  margin-bottom: 1.5rem;
  font-size: 0.875rem;
  text-align: left;
}

.result-details p {
  margin: 0.5rem 0;
  color: #555;
}

.results-canvas-container {
  margin-bottom: 1.5rem;
  display: flex;
  justify-content: center;
}

.results-canvas {
  width: 60%;
  max-width: 250px;
  height: auto;
  background: #f8f9fa;
  border-radius: 0.5rem;
  border: 2px solid #e0e0e0;
}

.result-stick {
  pointer-events: none;
}

.result-stick-border {
  pointer-events: none;
}

.share-message {
  margin-top: 1rem;
  color: #2ECC71;
  font-weight: 600;
}

@media (max-width: 480px) {
  .title {
    font-size: 1.5rem;
  }

  .subtitle {
    font-size: 0.875rem;
  }

  .game-container {
    padding: 0.5rem;
  }
}
</style>

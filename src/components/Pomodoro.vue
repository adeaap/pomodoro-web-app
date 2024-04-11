<script setup lang="ts">
import { ref, computed, onUpdated } from 'vue'
import Dialog from './Dialog.vue'

const pomodoroLength = ref(20) // In minutes
const shortBreakLength = ref(5) // In minutes
const longBreakLength = ref(15) // In minutes
const shortBreakHasHappened = ref(0)
const maxNumberOfShortBreaks = ref(1)
const timer = ref(0) // In seconds
const pauseTimer = ref(false)
const intervalId = ref<null | number>(null)
const settingsDialogVisible = ref(false)
const cicleState = ref<
  'default-state' | 'pomodoro' | 'short-break' | 'long-break' | 'pomodoro-cicle-complete'
>('default-state')

onUpdated(() => {
  if (
    cicleState.value === 'pomodoro' &&
    timer.value === pomodoroLength.value * 60 &&
    shortBreakHasHappened.value < maxNumberOfShortBreaks.value
  ) {
    setTimeout(() => alert('Time for a short break!'))
    resetTimer()

    cicleState.value = 'short-break'
    startTimer()
  }

  if (
    cicleState.value === 'short-break' &&
    timer.value === shortBreakLength.value * 60 &&
    shortBreakHasHappened.value <= maxNumberOfShortBreaks.value
  ) {
    shortBreakHasHappened.value += 1
    setTimeout(() => alert('Time to get back to work!'))
    resetTimer()

    cicleState.value = 'pomodoro'
    startTimer()
  }

  if (
    cicleState.value === 'pomodoro' &&
    timer.value === pomodoroLength.value * 60 &&
    shortBreakHasHappened.value === maxNumberOfShortBreaks.value
  ) {
    setTimeout(() => alert('Time for a long break!'))
    resetTimer()

    cicleState.value = 'long-break'
    startTimer()
  }

  if (cicleState.value === 'long-break' && timer.value === longBreakLength.value * 60) {
    shortBreakHasHappened.value = 0
    setTimeout(() => alert('Time to get back to work!'))
    resetTimer()

    cicleState.value = 'pomodoro'
    startTimer()
  }
})

const formattedTimer = computed(() => {
  const minutes = Math.floor(countDownTimer.value / 60)
  const seconds = countDownTimer.value % 60
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
})

const countDownTimer = computed(() => {
  if (cicleState.value === 'pomodoro' || cicleState.value === 'default-state') {
    return pomodoroLength.value * 60 - timer.value
  }
  if (cicleState.value === 'short-break') {
    return shortBreakLength.value * 60 - timer.value
  }
  if (cicleState.value === 'long-break') {
    return longBreakLength.value * 60 - timer.value
  }
  return 0
})

const resetTimer = () => {
  if (intervalId.value) {
    clearInterval(intervalId.value)
    timer.value = 0
  }
}

const startTimer = () => {
  intervalId.value = setInterval(() => {
    if (!pauseTimer.value) {
      timer.value += 1
    }
  }, 1000)
}
</script>

<template>
  <div class="pomodoro-app">
    <h1 class="title">The Pomodoro App</h1>
    <h3 class="sub-title">by Adea</h3>

    <div class="button-container">
      <button
        class="button"
        :class="{ button: true, selected: cicleState === 'pomodoro' }"
        @click="
          () => {
            resetTimer()
            cicleState = 'pomodoro'
            startTimer()
          }
        "
      >
        pomodoro
      </button>
      <button
        :class="{ button: true, selected: cicleState === 'short-break' }"
        @click="
          () => {
            if (shortBreakHasHappened >= maxNumberOfShortBreaks) {
              shortBreakHasHappened = 0
            }
            resetTimer()
            cicleState = 'short-break'
            startTimer()
          }
        "
      >
        short break
      </button>
      <button
        :class="{ button: true, selected: cicleState === 'long-break' }"
        @click="
          () => {
            resetTimer()
            cicleState = 'long-break'
            startTimer()
          }
        "
      >
        long break
      </button>
    </div>

    <!-- <h2>Cicle state: {{ cicleState }}</h2> -->

    <div class="time-circle">
      <p class="time">{{ formattedTimer }}</p>
      <br />
      <button
        v-if="cicleState === 'default-state'"
        class="startPauseButton"
        @click="
          () => {
            resetTimer()
            cicleState = 'pomodoro'
            startTimer()
          }
        "
      >
        START
      </button>
      <button
        v-else
        class="startPauseButton"
        @click="pauseTimer = !pauseTimer"
        v-text="pauseTimer ? 'RESUME' : 'PAUSE'"
      ></button>

      <div class="progress-circle">
        <circle-progress :percent="40" />
      </div>
    </div>
    <footer>
      <button class="button-dialog" @click="settingsDialogVisible = true">Settings</button>

      <Dialog :isVisible="settingsDialogVisible" @update:isVisible="settingsDialogVisible = $event">
        <div class="settings-form-container">
          <div>
            <input
              class="settings-input"
              type="number"
              id="pomodoroLength"
              v-model="pomodoroLength"
              min="1"
              max="60"
              step="1"
            />
            <label class="settings-details" for="pomodoroLength">Pomodoro length</label>
          </div>
          <div>
            <input
              class="settings-input"
              type="number"
              id="shortBreakLength"
              v-model="shortBreakLength"
              min="1"
              max="60"
              step="1"
            />
            <label class="settings-details" for="shortBreakLength">Short break length</label>
          </div>
          <div>
            <input
              class="settings-input"
              type="number"
              id="longBreakLength"
              v-model="longBreakLength"
              min="1"
              max="60"
              step="1"
            />

            <label class="settings-details" for="longBreakLength">Long break length</label>
          </div>
          <div>
            <input
              class="settings-input"
              type="number"
              id="maxNumberOfShortBreaks"
              v-model="maxNumberOfShortBreaks"
              min="1"
              max="60"
              step="1"
            />
            <label class="settings-details" for="maxNumberOfShortBreaks"
              >Max number of short breaks</label
            >
          </div>
        </div>
      </Dialog>
    </footer>
  </div>
</template>

<style scoped>
.pomodoro-app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.title {
  font-size: 2em;
  font: kumbh-sans !important;
  color: #d7e0ff;
}

.sub-title {
  font-size: 1e;
  font: kumbh-sans !important;
  color: #d7e0ff;
  margin-top: -10px;
}

.button.selected {
  font: kumbh-sans;
  border-radius: 30px;
  background-color: #f87060;
  color: #161932;
  font-weight: bold;
  border: none;
  padding: 10px 20px;
  margin-right: 10px;
  cursor: pointer;

  font-size: 12px;
  transition: background-color 0.3s;
}

.button {
  font: kumbh-sans;
  border-radius: 30px;
  background-color: #f87060;
  color: #161932;
  font-weight: bold;
  border: none;
  padding: 10px 20px;
  margin-right: 10px;
  cursor: pointer;

  text-transform: uppercase;

  font-size: 12px;
  transition: background-color 0.3s;
}
.button {
  font: kumbh-sans !important;
  border-radius: 30px;
  background-color: transparent;
  color: #d7e0ff;
  font-weight: bold;
  border: none;
  padding: 10px 20px;
  margin-right: 5px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
}

.button3 {
  font: kumbh-sans !important;
  border-radius: 30px;
  background-color: transparent;
  color: #d7e0ff;
  font-weight: bold;
  border: none;
  padding: 10px 20px;
  margin-right: 5px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
}
.button-container {
  margin-top: 30px;

  border-radius: 50px;
  width: fit-content;
  background-color: #161932;
  padding: 10px;
}

.startPauseButton {
  display: flex;
  background: none;
  cursor: pointer;
  border: none;
  font: kumbh-sans;
  color: #d7e0ff;
  font-size: 15px;
  font-weight: bold;
  letter-spacing: 15px;
  background-color: transparent;
  align-self: center;
  margin-top: -30px !important;
  margin-left: 20px;
}

.time-circle {
  margin-top: 50px;

  border-radius: 50%;
  width: 350px;
  height: 350px;
  background-color: #161932;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: 2em;
  color: #d7e0ff;
  box-shadow: -30px -30px 50px #272c5a;
}

.time {
  display: flex;
  align-self: center;
  margin-top: 20px;
  margin-bottom: 10px !important;
  font-size: 80px;
  background-color: transparent;
  font-weight: 600;
}

.reset {
  border-radius: 50px;
  background-color: #161932;
}

footer {
  padding-top: 16px;
}

label {
  font: kumbh-sans;
  color: #d7e0ff;
  font-size: 1.5em;
  margin-bottom: 10px;
  margin-right: 8px;
}

input {
  font: kumbh-sans;
  font-size: 1.5em;
  color: #d7e0ff;
  background-color: #161932;
  border: none;
  border-bottom: 1px solid #d7e0ff;
  padding: 5px;
  width: 50px;
  text-align: center;
}

.settings-details {
  font: kumbh-sans;
  color: #d7e0ff;
  font-size: 15px;
  margin-bottom: 10px;
  margin-right: 8px;
}

.settings-input {
  margin-right: 20px;
  font: kumbh-sans;
  font-size: 16px;
  color: #d7e0ff;
  background-color: #161932;
  border: none;
  border-bottom: 1px solid #d7e0ff;
  padding: 5px;
  width: 50px;
}

.settings-form-container {
  display: flex;
  gap: 12px;
  flex-direction: column;
}

.button-dialog {
  font: kumbh-sans;
  border-radius: 30px;
  background-color: #161932;
  color: #d7e0ff;
  font-weight: bold;
  border: none;
  padding: 10px 20px;
  margin-top: 20px;
  margin-right: 10px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
}
</style>

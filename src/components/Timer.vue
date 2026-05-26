<template>
  <div>
    <h1 :class="timerRunning || isPrepared ? 'running' : ''">{{ time }}</h1>
  </div>
</template>

<script>
function getTrimCount(count) {
  if (count < 5) return 0
  return count < 100 ? 1 : Math.ceil(count * 0.05)
}

function calculateAverage(values, count) {
  if (values.length < count) return '-'

  const lastValues = values.slice(-count)
  if (lastValues.some(value => !Number.isFinite(value))) return '-'

  const trimCount = getTrimCount(count)
  const sorted = lastValues.slice().sort((a, b) => a - b)
  const trimmed = sorted.slice(trimCount, sorted.length - trimCount)
  const sum = trimmed.reduce((acc, curr) => acc + curr, 0)

  return (sum / trimmed.length).toFixed(2)
}

export default {
  name: 'TimerComponent',
  data() {
    return {
      isPrepared: false,
      timerRunning: false,
      time: '0.00',
      initialTime: new Date(),
      finalTime: new Date(),
      bufferKey: '',
      timesBuffer: 0,
      times: [],
      intervalId: null
    }
  },
  methods: {
    handleKeyDown(e) {
      if (e.code === 'Space') e.preventDefault()

      if (this.timerRunning) {
        this.stopTimer()
        return
      }

      if (e.code === 'Space' && this.bufferKey === 'Space') {
        this.timesBuffer++
      } else {
        this.timesBuffer = 0
      }

      if (this.timesBuffer >= 8) {
        this.isPrepared = true
      }

      this.bufferKey = e.code
    },
    handleKeyUp(e) {
      if (e.code !== 'Space') return

      e.preventDefault()

      if (this.isPrepared && !this.timerRunning) {
        this.startTimer()
      }

      this.timesBuffer = 0
      this.bufferKey = ''
    },
    setBlinds() {
      document.addEventListener('keydown', this.handleKeyDown)
      document.addEventListener('keyup', this.handleKeyUp)

      this.intervalId = window.setInterval(() => {
        if (this.timerRunning) {
          this.finalTime = new Date()
          this.time = ((this.finalTime - this.initialTime) / 1000).toFixed(2)
        }
      }, 10)
    },
    startTimer() {
      this.initialTime = new Date()
      this.finalTime = new Date()
      this.time = '0.00'
      this.timerRunning = true
      this.isPrepared = false
    },
    stopTimer() {
      this.finalTime = new Date()
      this.time = ((this.finalTime - this.initialTime) / 1000).toFixed(2)
      this.timerRunning = false

      const solvedTime = this.time

      // Sync with localStorage before calculating averages.
      // This prevents using stale times after an old solve was deleted.
      this.loadTimes()
      this.time = solvedTime

      const newTime = {
        time: this.time,
        ao5: this.getAo(5),
        ao12: this.getAo(12),
        ao100: this.getAo(100)
      }

      this.times.push(newTime)
      localStorage.setItem('times', JSON.stringify(this.times))
      this.$emit('newScramble')
    },
    restartTimer() {
      this.isPrepared = false
      this.timerRunning = false
      this.time = '0.00'
      this.initialTime = new Date()
      this.finalTime = new Date()
      this.timesBuffer = 0
      this.bufferKey = ''
    },
    getAo(count) {
      const previousTimes = this.times
        .slice(-(count - 1))
        .map(t => parseFloat(t.time))

      const values = [...previousTimes, parseFloat(this.time)]
      return calculateAverage(values, count)
    },
    loadTimes() {
      const savedTimes = localStorage.getItem('times')
      this.times = savedTimes ? JSON.parse(savedTimes) : []
    }
  },
  mounted() {
    this.loadTimes()
    this.setBlinds()
  },
  beforeUnmount() {
    document.removeEventListener('keydown', this.handleKeyDown)
    document.removeEventListener('keyup', this.handleKeyUp)

    if (this.intervalId) {
      window.clearInterval(this.intervalId)
    }
  }
}
</script>

<style scoped>
@font-face {
  font-family: digi;
  src: url('@/assets/DS-DIGIT.TTF');
}

@font-face {
  font-family: digi2;
  src: url('@/assets/digital.mono.ttf');
}

h1 {
  font-family: monospace;
  font-size: 100px;
  cursor: default;
  user-select: none;
  text-align: center;
}

.running {
  color: #28a138;
  width: 100%;
  height: 100vh;
  position: absolute;
  background: rgb(var(--v-theme-background));
  margin: 0;
  top: 0;
  left: 0;
  font-size: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
}
</style>

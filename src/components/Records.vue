<template>
  <div class="resume-container">
    <div class="graphic-solves">
      <h3>Performance of the last 10 solves</h3>
      <v-sheet color="background">
        <v-sparkline
          color="secondary"
          line-width="3"
          :model-value="getLastSolves()"
          padding="20"
           :smooth="false"
          auto-draw
          stroke-linecap="round"
          :labels="getLastSolves()"
        />
      </v-sheet>
    </div>

    <div class="resume">
      <div class="ao-container">
        <span class="tittle ao">Current</span>
        <div class="ao-field">
          <div class="ao">Time:</div>
          <div class="ao">{{ getCurrentTime().time ?? '' }}</div>
        </div>
        <div class="ao-field">
          <div class="ao">Ao5:</div>
          <div class="ao">{{ getCurrentTime().ao5 ?? '' }}</div>
        </div>
        <div class="ao-field">
          <div class="ao">Ao12:</div>
          <div class="ao">{{ getCurrentTime().ao12 ?? '' }}</div>
        </div>
        <div class="ao-field">
          <div class="ao">Ao100:</div>
          <div class="ao">{{ getCurrentTime().ao100 ?? '' }}</div>
        </div>
      </div>

      <div class="ao-container">
        <span class="tittle ao">Best</span>
        <div class="ao-field">
          <div class="ao">Time:</div>
          <div class="ao">{{ getPB() }}</div>
        </div>
        <div class="ao-field">
          <div class="ao">Ao5:</div>
          <div class="ao">{{ getPBao(5) }}</div>
        </div>
        <div class="ao-field">
          <div class="ao">Ao12:</div>
          <div class="ao">{{ getPBao(12) }}</div>
        </div>
        <div class="ao-field">
          <div class="ao">Ao100:</div>
          <div class="ao">{{ getPBao(100) }}</div>
        </div>
      </div>
    </div>

    <div class="clear-btn-container">
      <v-btn rounded="lg" @click="clearTimes" color="primary" class="clear-times">Clear Times</v-btn>
    </div>
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

function recalculateTimes(times) {
  const solvedTimes = []

  return times.map(time => {
    solvedTimes.push(parseFloat(time.time))

    return {
      ...time,
      ao5: calculateAverage(solvedTimes, 5),
      ao12: calculateAverage(solvedTimes, 12),
      ao100: calculateAverage(solvedTimes, 100)
    }
  })
}

export default {
  name: 'RecordsTimes',
  props: {
    times: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    recalculatedTimes() {
      return recalculateTimes(this.times)
    }
  },
  methods: {
    getCurrentTime() {
      if (this.recalculatedTimes.length === 0) return {}
      return this.recalculatedTimes[this.recalculatedTimes.length - 1]
    },
    getPB() {
      const bestTime = this.times
        .map(t => parseFloat(t.time))
        .filter(value => Number.isFinite(value))
        .sort((a, b) => a - b)[0]

      return bestTime === undefined ? '-' : bestTime.toFixed(2)
    },
    deleteTime(index) {
      this.$emit('deleteTime', index)
    },
    clearTimes() {
      this.$emit('clearTimes')
    },
    getPBao(howMany) {
      const bestAo = this.recalculatedTimes
        .map(t => parseFloat(t[`ao${howMany}`]))
        .filter(value => Number.isFinite(value))
        .sort((a, b) => a - b)[0]

      return bestAo === undefined ? '-' : bestAo.toFixed(2)
    },
    getLastSolves() {
      return this.times
        .slice(-10)
        .map(t => parseFloat(t.time))
        .filter(value => Number.isFinite(value))
    }
  }
}
</script>

<style scoped>
.resume-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 30%;
  gap: 60px;
}

.resume {
  display: flex;
  flex-direction: row;
  width: 100%;
  margin: auto;
  justify-content: center;
}

.ao-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 45%;
}

.ao-field {
  width: 100%;
  display: flex;
}

.ao {
  width: 50%;
  font-size: 20px;
  text-align: end;
}

.ao2 {
  width: 50%;
  font-size: 20px;
  text-align: start;
}

.tittle {
  font-size: 24px;
  width: 100%;
  text-align: center;
  margin-bottom: 10px;
}

.table {
  display: block;
  height: auto;
  max-height: 80%;
}

.delete-btn {
  width: 15px;
  height: 15px;
  fill: #cacaca;
}

.delete-btn:hover {
  cursor: pointer;
  fill: #e4e4e4;
}

.graphic-solves {
  width: 100%;
  margin-top: 30px;
  height: 20%;
}

.clear-times {
  margin-top: 0;
}

h3 {
  text-align: center;
}
</style>

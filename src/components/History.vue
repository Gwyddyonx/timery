<template>
  <div class="history">
    <span id="history-title">Solves: {{ times.length }}</span>

    <v-table class="table" background="#fff">
      <thead>
        <tr>
          <th class="text-left">#</th>
          <th class="text-left">Time</th>
          <th class="text-left">Ao5</th>
          <th class="text-left">Ao12</th>
          <th class="text-left">Delete</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="time in displayedTimes" :key="time.originalIndex">
          <td>{{ time.solveNumber }}</td>
          <td>{{ time.time ?? '' }}</td>
          <td>{{ time.ao5 }}</td>
          <td>{{ time.ao12 }}</td>
          <td @click="deleteTime(time.originalIndex)" class="delete-td">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 384 512"
              class="delete-btn text-center"
            >
              <path
                d="M342.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L192 210.7 86.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L146.7 256 41.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L192 301.3 297.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L237.3 256 342.6 150.6z"
              />
            </svg>
          </td>
        </tr>
      </tbody>
    </v-table>
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

  return times.map((time, index) => {
    solvedTimes.push(parseFloat(time.time))

    return {
      ...time,
      originalIndex: index,
      solveNumber: index + 1,
      ao5: calculateAverage(solvedTimes, 5),
      ao12: calculateAverage(solvedTimes, 12),
      ao100: calculateAverage(solvedTimes, 100)
    }
  })
}

export default {
  name: 'HistoryTimes',
  props: {
    times: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    displayedTimes() {
      return recalculateTimes(this.times).slice().reverse()
    }
  },
  methods: {
    deleteTime(index) {
      this.$emit('deleteTime', index)
    },
    clearTimes() {
      this.$emit('clearTimes')
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
.resume {
  display: flex;
  flex-direction: row;
}

.history {
  height: 80%;
  display: flex;
  flex-direction: column;
}

.table {
  height: auto;
  overflow-y: auto;
}

.table div {
  overflow: hidden;
}

#history-title {
  font-size: 20px;
  margin: 20px;
}

.th1 {
  width: 30px;
}

.th-time {
  width: 70px;
}

.th-delete {
  width: 50px;
}

.tittle {
  font-size: 24px;
  width: 100%;
  text-align: center;
  margin-bottom: 10px;
}

.delete-btn {
  width: 15px;
  fill: #949494;
}

.delete-btn:hover {
  cursor: pointer;
  fill: #e4e4e4;
}

.resume-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.graphic-solves {
  width: 100%;
  margin-top: 30px;
  height: 20%;
}

.delete-td {
  display: flex;
  justify-content: center;
}

tbody,
thead {
  background: rgb(var(--v-theme-background));
}
</style>

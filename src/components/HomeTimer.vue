<template>

  <div class="content-container" color="primary">

    <Records :times="times" @delete-time="deleteTime" @clear-times="clearTimes" />
    <div class="timer-container">

      <Timer ref="timer" @new-scramble="newScramble" />

    </div>

    <div class="history-container">

      <History :times="times" @delete-time="deleteTime" @clear-times="clearTimes" />
    </div>

  </div>
</template>

<script>
import Timer from './Timer.vue';
import Utils from '../Services/Utils';
import Records from './Records.vue';
import History from './History.vue';

export default {
  name: 'HomeTimer',
  components: {
    Timer,
    Records
  },
  data() {
    return {
      scramble: "",
      times: [],
      toggle_theme: 0
    };
  },
  mounted() {
    this.newScramble();
    this.times = this.$refs.timer ? this.$refs.timer.times : [];
  },
  methods: {
    newScramble() {
      this.$emit("newScramble");
    },
    resetTimer() {
      this.$refs.timer.restartTimer();
    },
    clearTimes() {
      localStorage.removeItem('times');
      this.$refs.timer.times = [];
      this.times = this.$refs.timer.times;
      this.resetTimer()
    },
    deleteTime(index) {
      this.$refs.timer.times.splice(index, 1)
      localStorage.setItem('times', JSON.stringify(this.$refs.timer.times))
    }
  }
};
</script>

<style scoped>
.content-container {
  display: flex;
  height: 70vh;
  width: 100%;
  align-items: center;
  /* justify-content: center; */
  justify-content: flex-start;
}

h1 {
  font-family: monospace;
  font-size: 50px;
}

.title {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  cursor: default;
}

.scramble {
  font-size: 26px;
  cursor: default;
}

.clock {
  width: 50px;
  height: 50px;
  fill: #fff;
}

.timer-container {
  width: 40%;
}

.card-main {
  padding: 20px;
  display: flex;
  width: 80%;
}

.records-container {
  height: 80%;
}

.history-container {
  height: 100%;
  width: 25%;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
</style>

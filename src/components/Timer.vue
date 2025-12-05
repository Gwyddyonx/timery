<template>
  <div class="" >
    <h1 :class="timerRunning || isPrepared ? 'running' : ''">{{ time }}</h1>
  </div>
</template>

<script>

export default {
  name: 'TimerComponent',
  data: function () {
    return {
      isPrepared: false,
      timerRunning: false,
      time: 0.00,
      initialTime: new Date(),
      finalTime: new Date(),
      bufferKey: '',
      timesBuffer: 1,
      times: []
    }

  },
  methods: {
    setBlinds() {
      document.addEventListener('keydown', e => {
        if (this.timerRunning) {
          // Stop timer
          this.finalTimer = new Date()
          this.time = ((this.finalTime - this.initialTime) / 1000).toFixed(2)
          this.timerRunning = false

          let new_time = {
            time: this.time,
            ao5: this.getAo(5),
            ao12: this.getAo(12),
            ao100: this.getAo(100)
          }

          this.times.push(new_time)

          localStorage.setItem('times', JSON.stringify(this.times))
          this.$emit("newScramble")
          return
        }


        if (e.code == "Space" && this.bufferKey == "Space") {
          e.preventDefault();
          this.timesBuffer++
          // Starting timer
        } else {
          this.timesBuffer = 0
        }

        if (this.timesBuffer == 8) {
          this.isPrepared = true
        }

        this.bufferKey = e.code
      })

      document.addEventListener('keyup', e => {
        if (e.code == "Space") {
          e.preventDefault();
          // Starting timer
          if (this.isPrepared) {
            this.initialTime = new Date()
            this.timerRunning = true
            this.isPrepared = false
          }
        }
        this.timesBuffer = 0
      })


      window.setInterval(() => {
        if (this.timerRunning) {
          this.finalTime = new Date()
          this.time = ((this.finalTime - this.initialTime) / 1000).toFixed(2)
        }

      }, 10);

    },
    restartTimer() {
      this.isPrepared = false
      this.timerRunning = false
      this.time = 0.00
      this.initialTime = new Date()
      this.finalTime = new Date()
    },
    getAo(count) {
      if (this.times.length < count - 1) {
        return "-"
      }
    
      let lastSolves = this.times
        .toReversed()
        .slice(0, count - 1)
        .map(t => parseFloat(t.time))
    
      lastSolves.push(parseFloat(this.time))
    
      let trimCount = Math.floor(count * 0.05); 
      lastSolves.sort((a, b) => a - b);
    
      let trimmed = lastSolves.slice(trimCount, lastSolves.length - trimCount);
      let sum = trimmed.reduce((acc, curr) => acc + curr, 0);
      let avg = sum / trimmed.length;
    
      return avg.toFixed(2);
    }

  },
  mounted() {
    this.setBlinds()
    this.times = localStorage.getItem("times") != undefined ? JSON.parse(localStorage.getItem("times")) : []
  },
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
  /*font-family: digi;*/
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
  margin: 0px;
  top: 0;
  left: 0;
  font-size: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
}
</style>

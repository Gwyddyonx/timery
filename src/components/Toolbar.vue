<template>
  <v-toolbar color="primary" class="toolbar" height="150px">
    <!--v-app-bar-nav-icon></v-app-bar-nav-icon-->

    <div class="timery">
      <img src="../assets/logo.svg" class="logo" />
      <!--v-toolbar-title class="title">Timery</v-toolbar-title-->
      <h2 class="title">Timery</h2>
    </div>

    <div class="scramble-container">
      <h1 class="scramble">{{ scramble }}</h1>
      <v-btn rounded="lg" v-on:click="newScramble()" color="secondary" variant="outlined">New Scramble</v-btn>
    </div>


    <v-btn-toggle v-model="theme" mandatory shaped class="toggle-theme" @change="handleThemeChange"
      v-on:click="handleThemeChange" rounded="xl">
      <v-btn color="#211245">
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>

      <v-btn color="#388E3C">
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>

      <v-btn color="#1565C0">
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>

      <v-btn color="#546E7A">
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>

      <v-btn color="#009688">
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>

      <v-btn color="#3F51B5">
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>
    </v-btn-toggle>
  </v-toolbar>
</template>

<script>
import { useTheme } from 'vuetify'
import Utils from '../Services/Utils';

export default {
  data() {
    return {
      scramble: "",
      theme: 0,
      usetheme: null
    }
  },
  mounted() {
    this.usetheme = useTheme()
    this.newScramble();
    this.usetheme.global.name = localStorage.getItem("theme") ?? ""
    this.theme = localStorage.getItem("theme-id") ?? 0;
  },
  methods: {
    handleThemeChange(newThemeIndex) {
      const themeMapping = {
        0: 'purpleDarkTheme',
        1: 'greenDarkTheme',
        2: 'deepBlueDarkTheme',
        3: 'blueGreyDarkTheme',
        4: 'tealLightTheme',
        5: 'blueIndigoLightTheme'
      }
      this.usetheme.global.name = themeMapping[this.theme]
      localStorage.setItem("theme", themeMapping[this.theme])
      localStorage.setItem("theme-id", this.theme)
    },
    newScramble() {
      const util = new Utils();
      this.scramble = util.getScramble();
    }
  }
}
</script>
<style scoped>
.toggle-theme {
  margin-top: 20px;
  margin-right: 20px;
}

.logo {
  width: 40px;
  margin-left: 20px;
}

.title {
  width: min-content;
  margin: 20px;
}

.scramble-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: auto;
  margin: auto;
  height: 150px;
  cursor: default;
}

.scramble {
  font-family: monospace;
  font-size: 25px;
}

.timery {
  display: flex;
  width: 200px;
}
</style>

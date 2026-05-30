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
      <div class="scramble-actions">
        <v-btn rounded="lg" v-on:click="newScramble()" color="secondary" variant="outlined">New Scramble</v-btn>
        <v-btn
          icon="mdi-eye"
          color="secondary"
          variant="outlined"
          @click="showPreview = true"
          title="Previsualizar scramble"
        />
      </div>
    </div>

    <v-dialog v-model="showPreview" width="720">
      <v-card class="preview-dialog" color="background">
        <v-card-title class="preview-title">
          <span>Previsualizacion del scramble</span>
          <v-btn icon="mdi-close" color="secondary" variant="text" @click="showPreview = false" />
        </v-card-title>

        <v-card-text>
          <p class="preview-scramble">{{ scramble }}</p>
          <div class="cube-net">
            <div
              v-for="face in cubeFaces"
              :key="face.name"
              class="cube-face-wrapper"
            >
              <span class="face-label">{{ face.name }}</span>
              <div class="cube-face">
                <div
                  v-for="(color, index) in face.stickers"
                  :key="`${face.name}-${index}`"
                  class="sticker"
                  :style="{ backgroundColor: color }"
                />
              </div>
            </div>
          </div>
        </v-card-text>
      </v-card>
    </v-dialog>

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

const FACE_COLORS = {
  U: '#ffffff',
  R: '#d32f2f',
  F: '#2e7d32',
  D: '#fdd835',
  L: '#f57c00',
  B: '#1565c0'
}

const FACE_CONFIG = {
  U: { axis: 'y', layer: 1 },
  D: { axis: 'y', layer: -1 },
  R: { axis: 'x', layer: 1 },
  L: { axis: 'x', layer: -1 },
  F: { axis: 'z', layer: 1 },
  B: { axis: 'z', layer: -1 }
}

function createSolvedCube() {
  const stickers = []

  for (let x = -1; x <= 1; x++) {
    for (let z = -1; z <= 1; z++) {
      stickers.push({ color: FACE_COLORS.U, position: { x, y: 1, z }, normal: { x: 0, y: 1, z: 0 } })
      stickers.push({ color: FACE_COLORS.D, position: { x, y: -1, z }, normal: { x: 0, y: -1, z: 0 } })
    }
  }

  for (let x = -1; x <= 1; x++) {
    for (let y = -1; y <= 1; y++) {
      stickers.push({ color: FACE_COLORS.F, position: { x, y, z: 1 }, normal: { x: 0, y: 0, z: 1 } })
      stickers.push({ color: FACE_COLORS.B, position: { x, y, z: -1 }, normal: { x: 0, y: 0, z: -1 } })
    }
  }

  for (let z = -1; z <= 1; z++) {
    for (let y = -1; y <= 1; y++) {
      stickers.push({ color: FACE_COLORS.R, position: { x: 1, y, z }, normal: { x: 1, y: 0, z: 0 } })
      stickers.push({ color: FACE_COLORS.L, position: { x: -1, y, z }, normal: { x: -1, y: 0, z: 0 } })
    }
  }

  return stickers
}

function rotateVector(vector, axis, direction) {
  const { x, y, z } = vector

  if (axis === 'x') return direction === 1 ? { x, y: -z, z: y } : { x, y: z, z: -y }
  if (axis === 'y') return direction === 1 ? { x: z, y, z: -x } : { x: -z, y, z: x }

  return direction === 1 ? { x: -y, y: x, z } : { x: y, y: -x, z }
}

function applyMove(stickers, move) {
  const face = move[0]
  const modifier = move.slice(1)
  const config = FACE_CONFIG[face]
  if (!config) return

  const turns = modifier === '2' ? 2 : 1
  const clockwiseDirection = config.layer === 1 ? -1 : 1
  const direction = modifier === "'" ? -clockwiseDirection : clockwiseDirection

  for (let turn = 0; turn < turns; turn++) {
    stickers.forEach(sticker => {
      if (sticker.position[config.axis] !== config.layer) return

      sticker.position = rotateVector(sticker.position, config.axis, direction)
      sticker.normal = rotateVector(sticker.normal, config.axis, direction)
    })
  }
}

function sortFaceStickers(face, stickers) {
  const faceStickers = stickers.filter(sticker => {
    if (face === 'U') return sticker.normal.y === 1
    if (face === 'D') return sticker.normal.y === -1
    if (face === 'F') return sticker.normal.z === 1
    if (face === 'B') return sticker.normal.z === -1
    if (face === 'R') return sticker.normal.x === 1
    return sticker.normal.x === -1
  })

  return faceStickers
    .sort((a, b) => {
      const aRow = getFaceRow(face, a.position)
      const bRow = getFaceRow(face, b.position)
      const aColumn = getFaceColumn(face, a.position)
      const bColumn = getFaceColumn(face, b.position)

      return aRow - bRow || aColumn - bColumn
    })
    .map(sticker => sticker.color)
}

function getFaceRow(face, position) {
  if (face === 'U') return position.z + 1
  if (face === 'D') return 1 - position.z

  return 1 - position.y
}

function getFaceColumn(face, position) {
  if (face === 'B') return 1 - position.x
  if (face === 'R') return 1 - position.z
  if (face === 'L') return position.z + 1

  return position.x + 1
}

function getScrambledFaces(scramble) {
  const stickers = createSolvedCube()
  scramble
    .trim()
    .split(/\s+/)
    .filter(Boolean)
    .forEach(move => applyMove(stickers, move))

  return ['U', 'R', 'F', 'D', 'L', 'B'].map(face => ({
    name: face,
    stickers: sortFaceStickers(face, stickers)
  }))
}

export default {
  data() {
    return {
      scramble: "",
      theme: 0,
      usetheme: null,
      showPreview: false
    }
  },
  computed: {
    cubeFaces() {
      return getScrambledFaces(this.scramble)
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

.scramble-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.scramble {
  font-family: monospace;
  font-size: 25px;
}

.timery {
  display: flex;
  width: 200px;
}

.preview-dialog {
  border-radius: 8px;
  border: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
  box-shadow: 0 18px 50px rgba(0, 0, 0, 0.42);
}

.preview-title {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
  color: rgb(var(--v-theme-secondary));
  font-weight: 700;
}

.preview-scramble {
  margin: 0 0 18px;
  color: rgba(var(--v-theme-on-background), var(--v-medium-emphasis-opacity));
  font-family: monospace;
  font-size: 18px;
  text-align: center;
}

.cube-net {
  display: grid;
  grid-template-columns: repeat(3, minmax(120px, 1fr));
  gap: 18px;
}

.cube-face-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 14px 12px;
  border: 3px solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-radius: 8px;
}

.face-label {
  color: rgb(var(--v-theme-secondary));
  font-weight: 700;
}

.cube-face {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  width: 120px;
  aspect-ratio: 1;
  border: 2px solid rgba(0, 0, 0, 0.45);
  background: rgba(0, 0, 0, 0.45);
  gap: 2px;
}

.sticker {
  border: 1px solid rgba(0, 0, 0, 0.35);
  min-width: 0;
}

@media (max-width: 760px) {
  .cube-net {
    grid-template-columns: repeat(2, minmax(96px, 1fr));
  }

  .cube-face {
    width: 96px;
  }
}
</style>

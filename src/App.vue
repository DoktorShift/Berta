<script setup lang="ts">
import { ref } from 'vue';
import ModeSelection from './components/ModeSelection.vue';
import TouchDetection from './components/TouchDetection.vue';
import ResultDisplay from './components/ResultDisplay.vue';

type Mode = 'single' | 'pairs' | null;
type FingerData = { x: number; y: number; color: string; label: number };

const currentMode = ref<Mode>(null);
const showResult = ref(false);
const selectedFingers = ref<number[]>([]);
const fingerData = ref<FingerData[]>([]);
const detectionKey = ref(0);

const handleModeSelect = (mode: Mode) => {
  currentMode.value = mode;
};

const handleBackToMode = () => {
  currentMode.value = null;
  showResult.value = false;
  selectedFingers.value = [];
  fingerData.value = [];
};

const handleSelectionComplete = (fingers: number[], data: FingerData[]) => {
  selectedFingers.value = fingers;
  fingerData.value = data;
  showResult.value = true;
};

const handleBackToDetection = () => {
  showResult.value = false;
  selectedFingers.value = [];
  fingerData.value = [];
  detectionKey.value++;
};
</script>

<template>
  <div class="app">
    <main class="main" :class="{ fullscreen: currentMode || showResult }">
      <ModeSelection
        v-if="!currentMode"
        @select-mode="handleModeSelect"
      />

      <TouchDetection
        v-else-if="currentMode && !showResult"
        :key="detectionKey"
        :mode="currentMode"
        @back="handleBackToMode"
        @selection-complete="handleSelectionComplete"
      />

      <ResultDisplay
        v-else-if="showResult"
        :mode="currentMode!"
        :selected-fingers="selectedFingers"
        :finger-data="fingerData"
        @back="handleBackToDetection"
        @new-selection="handleBackToMode"
      />
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background: #fafafa;
  color: #000000;
  overflow: hidden;
}

.app {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #fafafa;
}

.main {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(1.5rem, 4vh, 3rem) clamp(1rem, 3vw, 2rem);
  overflow: hidden;
}

.main.fullscreen {
  padding: 0;
}
</style>

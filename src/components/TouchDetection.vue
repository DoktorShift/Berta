<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';

const props = defineProps<{
  mode: 'single' | 'pairs'
}>();

const emit = defineEmits<{
  back: []
  selectionComplete: [fingers: number[], positions: Array<{ x: number; y: number; color: string; label: number }>]
}>();

const touchArea = ref<HTMLElement | null>(null);
const touches = ref<Map<number, { x: number; y: number; color: string; label: number }>>(new Map());
const registeredTouches = ref<Array<{ x: number; y: number; color: string; label: number }>>([]);
const selectedIndices = ref<number[]>([]);
const isDetecting = ref(false);
const countdown = ref(5);
const stableCheckInterval = ref<number | null>(null);
const nextFingerLabel = ref(1);

const colors = [
  '#E74C3C',
  '#3498DB',
  '#2ECC71',
  '#F39C12',
  '#9B59B6',
  '#1ABC9C',
  '#E67E22',
  '#34495E',
  '#F1C40F',
  '#E91E63',
  '#16A085',
  '#C0392B',
  '#8E44AD',
  '#D35400',
  '#2980B9'
];

const MAX_PARTICIPANTS = 15;

const modeTitle = computed(() => {
  return props.mode === 'single' ? 'Eine Person' : 'Paare';
});

const getUniqueColor = (index: number) => {
  return colors[index % colors.length];
};

const fisherYatesShuffle = (array: number[]) => {
  const arr = [...array];
  for (let i = arr.length - 1; i > 0; i--) {
    const randomBytes = new Uint32Array(1);
    crypto.getRandomValues(randomBytes);
    const j = randomBytes[0] % (i + 1);
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
};

const startCountdown = () => {
  if (stableCheckInterval.value) {
    clearInterval(stableCheckInterval.value);
  }

  isDetecting.value = true;
  countdown.value = 5;

  stableCheckInterval.value = window.setInterval(() => {
    countdown.value--;

    if (countdown.value === 0) {
      if (stableCheckInterval.value) {
        clearInterval(stableCheckInterval.value);
        stableCheckInterval.value = null;
      }
      completeSelection();
    }
  }, 1000);
};

const handleTouchStart = (e: TouchEvent) => {
  e.preventDefault();

  const rect = touchArea.value?.getBoundingClientRect();
  if (!rect) return;

  Array.from(e.touches).forEach(touch => {
    if (!touches.value.has(touch.identifier) && registeredTouches.value.length < MAX_PARTICIPANTS) {
      const newTouch = {
        x: touch.clientX - rect.left,
        y: touch.clientY - rect.top,
        color: getUniqueColor(registeredTouches.value.length),
        label: nextFingerLabel.value++
      };
      touches.value.set(touch.identifier, newTouch);
      registeredTouches.value.push(newTouch);

      startCountdown();
    }
  });
};

const handleTouchEnd = (e: TouchEvent) => {
  e.preventDefault();

  Array.from(e.changedTouches).forEach(touch => {
    touches.value.delete(touch.identifier);
  });
};

const handleTouchMove = (e: TouchEvent) => {
  e.preventDefault();

  const rect = touchArea.value?.getBoundingClientRect();
  if (!rect) return;

  Array.from(e.touches).forEach(touch => {
    const existingTouch = touches.value.get(touch.identifier);
    if (existingTouch) {
      existingTouch.x = touch.clientX - rect.left;
      existingTouch.y = touch.clientY - rect.top;
    }
  });
};


const completeSelection = () => {
  const fingerCount = registeredTouches.value.length;
  const indices = Array.from({ length: fingerCount }, (_, i) => i);

  if (props.mode === 'single') {
    const shuffled = fisherYatesShuffle(indices);
    selectedIndices.value = [shuffled[0]];
  } else if (props.mode === 'pairs') {
    const shuffled = fisherYatesShuffle(indices);
    const half = Math.ceil(fingerCount / 2);
    selectedIndices.value = shuffled.slice(0, half);
  }

  isDetecting.value = false;
  emit('selectionComplete', selectedIndices.value, registeredTouches.value);
};

onMounted(() => {
  if (touchArea.value) {
    touchArea.value.addEventListener('touchstart', handleTouchStart, { passive: false });
    touchArea.value.addEventListener('touchend', handleTouchEnd, { passive: false });
    touchArea.value.addEventListener('touchmove', handleTouchMove, { passive: false });
  }
});

onUnmounted(() => {
  if (touchArea.value) {
    touchArea.value.removeEventListener('touchstart', handleTouchStart);
    touchArea.value.removeEventListener('touchend', handleTouchEnd);
    touchArea.value.removeEventListener('touchmove', handleTouchMove);
  }
  if (stableCheckInterval.value) {
    clearInterval(stableCheckInterval.value);
  }
});
</script>

<template>
  <div class="touch-detection">
    <div class="header">
      <button class="back-button" @click="emit('back')" aria-label="Zurück">
        <span class="back-icon">‹</span>
      </button>
      <h2 class="mode-title">{{ modeTitle }}</h2>
      <div v-if="isDetecting" class="countdown-badge">
        <div class="countdown-spinner"></div>
        <span class="countdown-text">{{ countdown }}</span>
      </div>
    </div>

    <div ref="touchArea" class="touch-area">
      <div
        v-for="(touch, index) in registeredTouches"
        :key="index"
        class="touch-point"
        :style="{
          left: `${touch.x}px`,
          top: `${touch.y}px`,
          background: touch.color
        }"
      >
        <div class="touch-label">{{ touch.label }}</div>
        <div class="touch-ring" :style="{ borderColor: touch.color }"></div>
        <div class="touch-ring-outer" :style="{ borderColor: touch.color }"></div>
      </div>

      <div v-if="registeredTouches.length === 0" class="placeholder">
        <div class="placeholder-text">Tippen um teilzunehmen</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.touch-detection {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #fafafa;
  position: fixed;
  top: 0;
  left: 0;
  animation: fadeIn 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: max(env(safe-area-inset-top), 1rem) 1.5rem 1rem;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 0.5px solid rgba(0, 0, 0, 0.06);
  position: relative;
  z-index: 10;
}

.back-button {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: transparent;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
  margin-right: 0.5rem;
}

.back-button:hover {
  background: rgba(0, 0, 0, 0.05);
}

.back-button:active {
  transform: scale(0.9);
}

.back-icon {
  font-size: 2rem;
  font-weight: 300;
  color: #000000;
  line-height: 1;
}

.mode-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #000000;
  letter-spacing: -0.02em;
  flex: 1;
  margin: 0;
}

.touch-area {
  flex: 1;
  position: relative;
  background: #ffffff;
  overflow: hidden;
  touch-action: none;
  user-select: none;
}

.touch-point {
  position: absolute;
  width: 90px;
  height: 90px;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  animation: touchAppear 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.16);
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

@keyframes touchAppear {
  from {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.3);
  }
  to {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}

.touch-label {
  font-size: 2rem;
  font-weight: 700;
  color: #ffffff;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  z-index: 2;
  letter-spacing: -0.02em;
}

.touch-ring {
  position: absolute;
  inset: -16px;
  border: 4px solid;
  border-radius: 50%;
  animation: touchPulse 2s cubic-bezier(0.16, 1, 0.3, 1) infinite;
  opacity: 0.5;
}

.touch-ring-outer {
  position: absolute;
  inset: -32px;
  border: 3px solid;
  border-radius: 50%;
  animation: touchPulseOuter 2s cubic-bezier(0.16, 1, 0.3, 1) infinite;
  opacity: 0.3;
}

@keyframes touchPulse {
  0%, 100% {
    transform: scale(1);
    opacity: 0.5;
  }
  50% {
    transform: scale(1.4);
    opacity: 0;
  }
}

@keyframes touchPulseOuter {
  0%, 100% {
    transform: scale(1);
    opacity: 0.3;
  }
  50% {
    transform: scale(1.6);
    opacity: 0;
  }
}

.placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}

.placeholder-text {
  font-size: clamp(1.125rem, 3vw, 1.5rem);
  color: #86868b;
  font-weight: 400;
  text-align: center;
  padding: 0 2rem;
  line-height: 1.4;
}

.hint-overlay {
  position: absolute;
  bottom: max(env(safe-area-inset-bottom), 3rem);
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1.5rem 2rem;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 20px;
  z-index: 5;
  animation: slideUp 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.hint-text {
  font-size: 1rem;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: -0.01em;
  white-space: nowrap;
}

.progress-bar {
  width: 200px;
  height: 4px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #ffffff;
  border-radius: 2px;
  transition: width 0.1s linear;
}

.countdown-badge {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 0.875rem;
  background: #000000;
  border-radius: 100px;
  animation: slideIn 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(10px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.countdown-spinner {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #ffffff;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.countdown-text {
  font-size: 0.9375rem;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: -0.01em;
  min-width: 0.75rem;
  text-align: center;
}

@media (max-width: 640px) {
  .touch-point {
    width: 80px;
    height: 80px;
  }

  .touch-label {
    font-size: 1.75rem;
  }

  .hint-overlay {
    bottom: max(env(safe-area-inset-bottom), 2rem);
    padding: 1.25rem 1.5rem;
  }

  .hint-text {
    font-size: 0.9375rem;
  }

  .progress-bar {
    width: 160px;
  }
}
</style>

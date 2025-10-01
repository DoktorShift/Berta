<script setup lang="ts">
import { computed } from 'vue';

type FingerData = { x: number; y: number; color: string; label: number };

const props = defineProps<{
  mode: 'single' | 'pairs'
  selectedFingers: number[]
  fingerData: FingerData[]
}>();

const emit = defineEmits<{
  back: []
  newSelection: []
}>();

const resultData = computed(() => {
  if (props.mode === 'single') {
    const winnerIndex = props.selectedFingers[0];
    const winner = props.fingerData[winnerIndex];

    return {
      winner
    };
  } else {
    const team1Indices = props.selectedFingers;
    const team2Indices = Array.from(
      { length: props.fingerData.length },
      (_, i) => i
    ).filter(i => !team1Indices.includes(i));

    const team1 = team1Indices.map(i => props.fingerData[i]);
    const team2 = team2Indices.map(i => props.fingerData[i]);

    return {
      team1,
      team2
    };
  }
});
</script>

<template>
  <div class="result-display">
    <div class="header">
      <button class="back-button" @click="emit('newSelection')" aria-label="Zurück">
        <span class="back-icon">‹</span>
      </button>
    </div>

    <div class="result-content">
      <div v-if="mode === 'single' && resultData.winner" class="single-result">
        <div class="winner-card">
          <div class="winner-badge">Gewinner</div>
          <div
            class="winner-circle"
            :style="{ background: resultData.winner.color }"
          >
            <div class="winner-number">{{ resultData.winner.label }}</div>
          </div>
        </div>

        <div class="divider"></div>

        <div class="all-participants">
          <div class="section-label">{{ fingerData.length }} Teilnehmer</div>
          <div class="participants-row">
            <div
              v-for="finger in fingerData"
              :key="finger.label"
              class="participant-mini"
              :class="{ 'is-winner': finger.label === resultData.winner.label }"
              :style="{ background: finger.color }"
            >
              <div class="mini-number">{{ finger.label }}</div>
            </div>
          </div>
        </div>
      </div>

      <div v-else class="pairs-result">
        <div class="teams-container">
          <div class="team-section">
            <div class="team-label">Team 1</div>
            <div class="team-members-row">
              <div
                v-for="(finger, idx) in resultData.team1"
                :key="finger.label"
                class="team-member-circle"
                :style="{
                  background: finger.color,
                  animationDelay: `${idx * 0.05}s`
                }"
              >
                <div class="member-num">{{ finger.label }}</div>
              </div>
            </div>
          </div>

          <div class="vs-separator">
            <div class="vs-line"></div>
            <div class="vs-badge">VS</div>
            <div class="vs-line"></div>
          </div>

          <div class="team-section">
            <div class="team-label">Team 2</div>
            <div class="team-members-row">
              <div
                v-for="(finger, idx) in resultData.team2"
                :key="finger.label"
                class="team-member-circle"
                :style="{
                  background: finger.color,
                  animationDelay: `${idx * 0.05}s`
                }"
              >
                <div class="member-num">{{ finger.label }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="footer">
      <button class="footer-button" @click="emit('back')">
        <span class="button-content">
          <span class="button-icon">↻</span>
          <span>Wiederholen</span>
        </span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.result-display {
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
  padding: max(env(safe-area-inset-top), 1rem) 1.5rem 1rem;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 0.5px solid rgba(0, 0, 0, 0.06);
  z-index: 10;
}

.back-button {
  width: 40px;
  height: 40px;
  border: none;
  border-radius: 12px;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
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

.result-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem 1.5rem 1rem;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

.single-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
  width: 100%;
  max-width: 420px;
  animation: slideUp 0.5s cubic-bezier(0.16, 1, 0.3, 1);
  margin-bottom: 1rem;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.winner-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  width: 100%;
}

.winner-badge {
  font-size: 0.8125rem;
  font-weight: 600;
  color: #000000;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  opacity: 0.4;
}

.winner-circle {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.12),
    0 0 0 8px rgba(255, 255, 255, 0.4);
  animation: scaleIn 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes scaleIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.winner-number {
  font-size: 5rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.02em;
}

.divider {
  width: 100%;
  height: 1px;
  background: linear-gradient(to right, transparent, rgba(0, 0, 0, 0.1), transparent);
}

.all-participants {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.section-label {
  font-size: 0.9375rem;
  font-weight: 600;
  color: #000000;
  opacity: 0.5;
  text-align: center;
  letter-spacing: -0.01em;
}

.participants-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.75rem;
}

.participant-mini {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
  animation: popIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) backwards;
  opacity: 0.6;
}

@keyframes popIn {
  from {
    opacity: 0;
    transform: scale(0);
  }
  to {
    opacity: 0.6;
    transform: scale(1);
  }
}

.participant-mini:nth-child(1) { animation-delay: 0.05s; }
.participant-mini:nth-child(2) { animation-delay: 0.08s; }
.participant-mini:nth-child(3) { animation-delay: 0.11s; }
.participant-mini:nth-child(4) { animation-delay: 0.14s; }
.participant-mini:nth-child(5) { animation-delay: 0.17s; }
.participant-mini:nth-child(6) { animation-delay: 0.20s; }
.participant-mini:nth-child(7) { animation-delay: 0.23s; }
.participant-mini:nth-child(8) { animation-delay: 0.26s; }
.participant-mini:nth-child(9) { animation-delay: 0.29s; }
.participant-mini:nth-child(10) { animation-delay: 0.32s; }
.participant-mini:nth-child(11) { animation-delay: 0.35s; }
.participant-mini:nth-child(12) { animation-delay: 0.38s; }
.participant-mini:nth-child(13) { animation-delay: 0.41s; }
.participant-mini:nth-child(14) { animation-delay: 0.44s; }
.participant-mini:nth-child(15) { animation-delay: 0.47s; }

.participant-mini.is-winner {
  opacity: 1;
  transform: scale(1);
  border: 3px solid #000000;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

.mini-number {
  font-size: 1.5rem;
  font-weight: 700;
  color: #ffffff;
}

.pairs-result {
  width: 100%;
  max-width: 600px;
  animation: slideUp 0.5s cubic-bezier(0.16, 1, 0.3, 1);
  margin-bottom: 1rem;
}

.teams-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.team-section {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.team-label {
  font-size: 1.5rem;
  font-weight: 700;
  color: #000000;
  letter-spacing: -0.02em;
}

.team-members-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.team-member-circle {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  animation: popIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) backwards;
}

.member-num {
  font-size: 1.75rem;
  font-weight: 700;
  color: #ffffff;
}

.vs-separator {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin: 0.5rem 0;
}

.vs-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(to right, transparent, rgba(0, 0, 0, 0.1), transparent);
}

.vs-badge {
  font-size: 0.875rem;
  font-weight: 700;
  color: #000000;
  opacity: 0.3;
  letter-spacing: 0.1em;
}

.footer {
  padding: 1rem 1.5rem max(env(safe-area-inset-bottom), 1rem);
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-top: 0.5px solid rgba(0, 0, 0, 0.06);
  flex-shrink: 0;
}

.footer-button {
  width: 100%;
  height: 56px;
  border: none;
  border-radius: 14px;
  background: #000000;
  color: #ffffff;
  font-size: 1.0625rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
  letter-spacing: -0.01em;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
}

.footer-button:hover {
  transform: scale(0.98);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

.footer-button:active {
  transform: scale(0.96);
}

.button-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.button-icon {
  font-size: 1.25rem;
}

@media (max-width: 640px) {
  .result-content {
    padding: 1rem 1rem 0.5rem;
    align-items: flex-start;
  }

  .single-result {
    gap: 1.5rem;
    padding-top: 0.5rem;
  }

  .pairs-result {
    padding-top: 0.5rem;
  }

  .teams-container {
    gap: 1.5rem;
  }

  .winner-card {
    gap: 1rem;
  }

  .winner-circle {
    width: 140px;
    height: 140px;
  }

  .winner-number {
    font-size: 3.5rem;
  }

  .team-member-circle {
    width: 52px;
    height: 52px;
  }

  .member-num {
    font-size: 1.375rem;
  }

  .participant-mini {
    width: 48px;
    height: 48px;
  }

  .mini-number {
    font-size: 1.25rem;
  }

  .all-participants {
    gap: 0.875rem;
  }

  .participants-row {
    gap: 0.625rem;
  }

  .footer {
    padding: 0.75rem 1rem max(env(safe-area-inset-bottom), 0.75rem);
  }

  .footer-button {
    height: 50px;
    font-size: 1rem;
    border-radius: 12px;
  }
}
</style>

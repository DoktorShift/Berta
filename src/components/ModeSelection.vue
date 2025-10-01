<script setup lang="ts">
const emit = defineEmits<{
  selectMode: [mode: 'single' | 'pairs']
}>();

const modes = [
  {
    id: 'single',
    title: 'Eine Person',
    description: 'Wählt zufällig eine Person aus',
    symbol: '1'
  },
  {
    id: 'pairs',
    title: 'Paare',
    description: 'Teilt in zwei Teams auf',
    symbol: '2'
  }
];

const handleSupportClick = () => {
  window.location.href = 'lightning:LNURL1DP68GURN8GHJ7ARFD4JKXCT5VD5X2U3WD3HXY6T5WVHXGEF0D3H82UNVWQHKU4ECG9XHQY9G5JR';
};
</script>

<template>
  <div class="mode-selection">
    <div class="logo-container">
      <img src="/RandomBerta.png" alt="R&B Logo" class="app-logo" />
    </div>

    <div class="intro">
      <p class="subtitle">Lege die Finger aller Teilnehmer auf den Bildschirm</p>
    </div>

    <div class="modes">
      <button
        v-for="mode in modes"
        :key="mode.id"
        class="mode-card"
        @click="emit('selectMode', mode.id as 'single' | 'pairs')"
      >
        <div class="mode-symbol">{{ mode.symbol }}</div>
        <div class="mode-info">
          <h3 class="mode-title">{{ mode.title }}</h3>
          <p class="mode-description">{{ mode.description }}</p>
        </div>
      </button>
    </div>

    <div class="footer-info">
      <a
        href="https://github.com/DoktorShift"
        target="_blank"
        rel="noopener noreferrer"
        class="profile-link"
        aria-label="GitHub Profile"
      >
        <img
          src="https://avatars.githubusercontent.com/u/106493492?v=4"
          alt="DoktorShift"
          class="profile-avatar"
        />
      </a>

      <button
        @click="handleSupportClick"
        class="support-badge"
        aria-label="Support via Lightning"
      >
        <svg class="lightning-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M13 2L3 14h8l-1 8 10-12h-8l1-8z" fill="currentColor"/>
        </svg>
        <span class="support-text">Support</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.mode-selection {
  width: 100%;
  max-width: 600px;
  display: flex;
  flex-direction: column;
  gap: clamp(2rem, 5vh, 4rem);
  animation: fadeIn 0.6s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(24px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.logo-container {
  display: flex;
  justify-content: center;
  animation: fadeIn 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

.app-logo {
  width: clamp(80px, 15vw, 120px);
  height: clamp(80px, 15vw, 120px);
  border-radius: 50%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

.app-logo:hover {
  transform: scale(1.05) rotate(5deg);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.2);
}

.intro {
  text-align: center;
}

.title {
  font-size: clamp(1.75rem, 4vw, 2.25rem);
  font-weight: 600;
  color: #000000;
  letter-spacing: -0.03em;
  line-height: 1.2;
  margin-bottom: 0.5rem;
}

.subtitle {
  font-size: clamp(0.9375rem, 2.5vw, 1.0625rem);
  color: #86868b;
  line-height: 1.4;
  font-weight: 400;
}

.modes {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.mode-card {
  background: #ffffff;
  border: none;
  border-radius: 16px;
  padding: clamp(1.25rem, 3vw, 1.75rem) clamp(1.5rem, 4vw, 2rem);
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  display: flex;
  align-items: center;
  gap: clamp(1rem, 3vw, 1.5rem);
  text-align: left;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
}

.mode-card:hover {
  transform: scale(1.02);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
}

.mode-card:active {
  transform: scale(0.98);
  transition: all 0.1s cubic-bezier(0.16, 1, 0.3, 1);
}

.mode-symbol {
  width: clamp(56px, 12vw, 72px);
  height: clamp(56px, 12vw, 72px);
  border-radius: 14px;
  background: #f5f5f7;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: clamp(1.5rem, 4vw, 2rem);
  font-weight: 600;
  color: #000000;
  flex-shrink: 0;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.mode-card:hover .mode-symbol {
  background: #000000;
  color: #ffffff;
}

.mode-info {
  flex: 1;
  min-width: 0;
}

.mode-title {
  font-size: clamp(1.125rem, 3vw, 1.375rem);
  font-weight: 600;
  color: #000000;
  letter-spacing: -0.02em;
  margin-bottom: 4px;
  line-height: 1.3;
}

.mode-description {
  font-size: clamp(0.875rem, 2.5vw, 1rem);
  color: #86868b;
  line-height: 1.4;
  font-weight: 400;
}

@media (max-width: 640px) {
  .mode-card {
    padding: 1.25rem 1.5rem;
  }
}

.footer-info {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  padding-top: clamp(1rem, 3vh, 2rem);
  animation: fadeIn 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.3s backwards;
}

.profile-link {
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  border-radius: 50%;
}

.profile-link:hover {
  transform: scale(1.1);
}

.profile-link:active {
  transform: scale(0.95);
}

.profile-avatar {
  width: clamp(48px, 10vw, 56px);
  height: clamp(48px, 10vw, 56px);
  border-radius: 50%;
  border: 2px solid rgba(0, 0, 0, 0.08);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.profile-link:hover .profile-avatar {
  border-color: rgba(0, 0, 0, 0.15);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.support-badge {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: clamp(10px, 2vw, 12px) clamp(14px, 3vw, 18px);
  background: linear-gradient(135deg, #FFA500 0%, #FF8C00 100%);
  color: #ffffff;
  border: none;
  border-radius: 50px;
  font-size: clamp(0.875rem, 2vw, 1rem);
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  box-shadow: 0 4px 16px rgba(255, 165, 0, 0.3);
  letter-spacing: -0.01em;
}

.support-badge:hover {
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 8px 24px rgba(255, 165, 0, 0.4);
}

.support-badge:active {
  transform: translateY(0) scale(0.98);
}

.lightning-icon {
  width: clamp(16px, 3.5vw, 20px);
  height: clamp(16px, 3.5vw, 20px);
  flex-shrink: 0;
}

.support-text {
  line-height: 1;
}

@media (max-width: 480px) {
  .footer-info {
    gap: 0.75rem;
  }

  .profile-avatar {
    width: 48px;
    height: 48px;
  }

  .support-badge {
    padding: 10px 14px;
  }
}
</style>

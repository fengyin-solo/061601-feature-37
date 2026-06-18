<script setup lang="ts">
import { computed } from 'vue'
import { useGameStore } from '../stores/gameStore'

const gameStore = useGameStore()

const reward = computed(() => gameStore.currentReward)

function closeModal() {
  gameStore.closeRewardModal()
}
</script>

<template>
  <Teleport to="body">
    <Transition name="fade">
      <div v-if="gameStore.showRewardModal && reward" class="modal-overlay" @click.self="closeModal">
        <div class="modal-content reward-modal">
          <div class="reward-header">
            <div class="reward-icon">{{ reward.icon }}</div>
            <div class="reward-badge">🏆 里程碑达成</div>
          </div>

          <h2 class="reward-title">{{ reward.name }}</h2>
          
          <p class="reward-description">{{ reward.description }}</p>

          <div class="reward-details">
            <div v-if="reward.resourceReward" class="reward-item">
              <span class="reward-item-icon">💰</span>
              <span class="reward-item-text">代币 +{{ reward.resourceReward }}</span>
            </div>
            <div v-if="reward.cardReward" class="reward-item">
              <span class="reward-item-icon">🎴</span>
              <span class="reward-item-text">神秘卡牌 x1</span>
            </div>
          </div>

          <div class="collection-progress">
            <span class="progress-label">当前收藏进度</span>
            <div class="progress-bar">
              <div 
                class="progress-fill" 
                :style="{ width: `${gameStore.collectionRatePercent}%` }"
              ></div>
            </div>
            <span class="progress-text">{{ gameStore.collectionRatePercent }}%</span>
          </div>

          <button class="btn btn-primary confirm-btn" @click="closeModal">
            太棒了！
          </button>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.reward-modal {
  padding: 32px;
  max-width: 400px;
  text-align: center;
  animation: bounceIn 0.5s ease-out;
}

@keyframes bounceIn {
  0% {
    transform: scale(0.3);
    opacity: 0;
  }
  50% {
    transform: scale(1.05);
  }
  70% {
    transform: scale(0.9);
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.reward-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}

.reward-icon {
  font-size: 64px;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

.reward-badge {
  padding: 6px 16px;
  background: linear-gradient(135deg, #f59e0b, #ef4444);
  color: white;
  border-radius: 9999px;
  font-size: 13px;
  font-weight: 600;
}

.reward-title {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 12px;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.reward-description {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.6;
  margin-bottom: 20px;
}

.reward-details {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 24px;
  padding: 16px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-md);
}

.reward-item {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 15px;
  font-weight: 500;
  color: var(--text-primary);
}

.reward-item-icon {
  font-size: 20px;
}

.collection-progress {
  margin-bottom: 24px;
}

.progress-label {
  display: block;
  font-size: 12px;
  color: var(--text-secondary);
  margin-bottom: 8px;
}

.progress-bar {
  height: 8px;
  background: var(--bg-secondary);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 6px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--accent-primary), var(--accent-secondary));
  border-radius: 4px;
  transition: width 0.5s ease;
}

.progress-text {
  font-size: 12px;
  font-weight: 600;
  color: var(--accent-primary);
}

.confirm-btn {
  width: 100%;
  padding: 12px;
  font-size: 15px;
  font-weight: 600;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

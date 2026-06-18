<script setup lang="ts">
import { computed } from 'vue'
import { useGameStore } from '../stores/gameStore'
import { getTimeLabel, getTimeIcon } from '../utils/gameUtils'
import gameConfig from '../config/gameConfig'

const emit = defineEmits<{
  (e: 'toggle-save'): void
  (e: 'toggle-cards'): void
  (e: 'toggle-history'): void
  (e: 'toggle-theme'): void
  (e: 'reset'): void
}>()

const gameStore = useGameStore()

const daysRemaining = computed(() => 
  Math.max(0, gameConfig.totalDays - gameStore.day + 1)
)
</script>

<template>
  <header class="top-bar card">
    <div class="game-title">
      <span class="title-icon">💝</span>
      <h1>恋爱物语</h1>
    </div>

    <div class="status-info">
      <div class="status-item day">
        <span class="status-icon">📅</span>
        <span>第 {{ gameStore.day }}/{{ gameConfig.totalDays }} 天</span>
      </div>
      <div class="status-item time">
        <span class="status-icon">{{ getTimeIcon(gameStore.timeSlot) }}</span>
        <span>{{ getTimeLabel(gameStore.timeSlot) }}</span>
      </div>
      <div class="status-item actions">
        <span class="status-icon">⚡</span>
        <span>行动力 {{ gameStore.actionsRemaining }}</span>
      </div>
      <div class="status-item resources">
        <span class="status-icon">💰</span>
        <span>{{ gameStore.resources }} 代币</span>
      </div>
      <div class="status-item collection" @click.stop="emit('toggle-cards')">
        <span class="status-icon">🎴</span>
        <span>{{ gameStore.collectionRatePercent }}% 收藏</span>
      </div>
    </div>

    <div class="event-hint-bar">
      <span class="hint-text">{{ gameStore.eventHint }}</span>
    </div>

    <div class="toolbar">
      <button class="toolbar-btn" @click="emit('toggle-cards')" title="卡牌收藏">
        🎴
      </button>
      <button class="toolbar-btn" @click="emit('toggle-history')" title="历史记录">
        📜
      </button>
      <button class="toolbar-btn" @click="emit('toggle-save')" title="存档/读档">
        💾
      </button>
      <button class="toolbar-btn" @click="emit('toggle-theme')" title="切换主题">
        {{ gameStore.darkMode ? '☀️' : '🌙' }}
      </button>
      <button class="toolbar-btn reset" @click="emit('reset')" title="重新开始">
        🔄
      </button>
    </div>
  </header>
</template>

<style scoped>
.top-bar {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  gap: 20px;
}

.game-title {
  display: flex;
  align-items: center;
  gap: 10px;
}

.title-icon {
  font-size: 28px;
}

.game-title h1 {
  font-size: 20px;
  font-weight: 700;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.status-info {
  display: flex;
  gap: 20px;
  flex: 1;
  justify-content: center;
}

.status-item {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-md);
  font-size: 14px;
  font-weight: 500;
  cursor: default;
  transition: all 0.2s;
}

.status-item.collection {
  cursor: pointer;
  background: var(--accent-light);
  color: var(--accent-primary);
}

.status-item.collection:hover {
  background: var(--accent-primary);
  color: white;
}

.status-icon {
  font-size: 18px;
}

.event-hint-bar {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-top: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 20px;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  color: white;
  border-radius: 9999px;
  font-size: 13px;
  font-weight: 500;
  white-space: nowrap;
  z-index: 10;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  animation: slideDown 0.3s ease-out;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.event-hint-bar .hint-text {
  font-size: 13px;
}

.toolbar {
  display: flex;
  gap: 8px;
}

.toolbar-btn {
  width: 40px;
  height: 40px;
  border-radius: var(--radius-md);
  background: var(--bg-tertiary);
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.toolbar-btn:hover {
  background: var(--accent-light);
  transform: scale(1.05);
}

.toolbar-btn.reset:hover {
  background: #fee2e2;
}

@media (max-width: 768px) {
  .top-bar {
    flex-wrap: wrap;
  }
  
  .status-info {
    order: 3;
    width: 100%;
    justify-content: space-around;
    gap: 8px;
    flex-wrap: wrap;
  }
  
  .status-item {
    padding: 6px 10px;
    font-size: 12px;
  }
  
  .game-title h1 {
    font-size: 18px;
  }

  .event-hint-bar {
    position: static;
    transform: none;
    order: 4;
    width: 100%;
    justify-content: center;
    margin-top: 8px;
    font-size: 12px;
    animation: none;
  }
}
</style>

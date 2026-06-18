<script setup lang="ts">
import { computed } from 'vue'
import { useGameStore } from '../stores/gameStore'
import gameConfig from '../config/gameConfig'
import {
  getAffinityColor,
  getAffinityStage,
  getMoodColor,
  getMoodLabel,
  getRarityLabel,
  getRarityColor
} from '../utils/gameUtils'

const gameStore = useGameStore()

const charactersWithConfig = computed(() => {
  return gameStore.unlockedCharacters.map(charState => {
    const config = gameConfig.characters.find(c => c.id === charState.id)
    const collectionRate = gameStore.getCharacterCollectionRate(charState.id)
    const nextCard = gameStore.getNextCardForCharacter(charState.id)
    const charCards = gameConfig.cards.filter(c => c.characterId === charState.id)
    const collectedCount = charCards.filter(c => gameStore.collectedCards.includes(c.id)).length
    return { 
      state: charState, 
      config, 
      collectionRate,
      nextCard,
      collectedCount,
      totalCount: charCards.length
    }
  }).filter(item => item.config)
})

function selectCharacter(id: string) {
  gameStore.selectCharacter(id)
}
</script>

<template>
  <div class="character-panel card">
    <h2 class="panel-title">
      <span class="title-icon">💕</span>
      角色状态
    </h2>

    <div class="character-list">
      <div
        v-for="item in charactersWithConfig"
        :key="item.state.id"
        class="character-card"
        :class="{ selected: gameStore.selectedCharacterId === item.state.id }"
        @click="selectCharacter(item.state.id)"
      >
        <div class="character-avatar">{{ item.config?.avatar }}</div>
        
        <div class="character-info">
          <div class="character-header">
            <span class="character-name">{{ item.config?.name }}</span>
            <span class="affinity-stage">{{ getAffinityStage(item.state.affinity) }}</span>
          </div>
          
          <div class="stat-row">
            <span class="stat-label">好感</span>
            <div class="progress-bar">
              <div 
                class="progress-fill"
                :style="{ 
                  width: `${Math.max(0, (item.state.affinity / gameConfig.maxAffinity) * 100)}%`,
                  backgroundColor: getAffinityColor(item.state.affinity, gameConfig.maxAffinity)
                }"
              ></div>
            </div>
            <span class="stat-value">{{ item.state.affinity }}</span>
          </div>
          
          <div class="stat-row">
            <span class="stat-label">心情</span>
            <div class="progress-bar">
              <div 
                class="progress-fill"
                :style="{ 
                  width: `${(item.state.mood / gameConfig.maxMood) * 100}%`,
                  backgroundColor: getMoodColor(item.state.mood)
                }"
              ></div>
            </div>
            <span class="stat-value mood" :style="{ color: getMoodColor(item.state.mood) }">
              {{ getMoodLabel(item.state.mood) }}
            </span>
          </div>

          <div class="stat-row collection-row">
            <span class="stat-label">收藏</span>
            <div class="progress-bar">
              <div 
                class="progress-fill collection-fill"
                :style="{ width: `${item.collectionRate * 100}%` }"
              ></div>
            </div>
            <span class="stat-value collection-value">
              {{ item.collectedCount }}/{{ item.totalCount }}
            </span>
          </div>

          <div v-if="item.nextCard" class="next-card-hint">
            <span class="hint-icon">💡</span>
            <span class="hint-text">
              下一张：
              <span class="next-card-rarity" :style="{ color: getRarityColor(item.nextCard.rarity) }">
                【{{ getRarityLabel(item.nextCard.rarity) }}】
              </span>
              ???
            </span>
          </div>
        </div>
      </div>
    </div>

    <div v-if="gameStore.currentCharacterConfig" class="character-detail">
      <div class="detail-divider"></div>
      <p class="detail-description">{{ gameStore.currentCharacterConfig.description }}</p>
      <p class="detail-personality">性格：{{ gameStore.currentCharacterConfig.personality }}</p>
    </div>
  </div>
</template>

<style scoped>
.character-panel {
  padding: 20px;
}

.panel-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.title-icon {
  font-size: 22px;
}

.character-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.character-card {
  display: flex;
  gap: 14px;
  padding: 14px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-md);
  cursor: pointer;
  transition: all 0.2s;
  border: 2px solid transparent;
}

.character-card:hover {
  transform: translateX(4px);
  background: var(--accent-light);
}

.character-card.selected {
  border-color: var(--accent-primary);
  background: var(--accent-light);
}

.character-avatar {
  width: 56px;
  height: 56px;
  background: var(--bg-secondary);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  flex-shrink: 0;
}

.character-info {
  flex: 1;
  min-width: 0;
}

.character-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.character-name {
  font-weight: 600;
  font-size: 15px;
}

.affinity-stage {
  font-size: 12px;
  padding: 2px 8px;
  background: var(--accent-primary);
  color: white;
  border-radius: 9999px;
}

.stat-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 12px;
  color: var(--text-secondary);
  width: 30px;
  flex-shrink: 0;
}

.progress-bar {
  flex: 1;
  height: 6px;
  background: var(--bg-secondary);
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 3px;
  transition: width 0.3s;
}

.stat-value {
  font-size: 12px;
  font-weight: 500;
  min-width: 30px;
  text-align: right;
}

.stat-value.mood {
  font-size: 11px;
}

.collection-row {
  margin-top: 6px;
}

.collection-fill {
  background: linear-gradient(90deg, var(--accent-primary), var(--accent-secondary));
}

.collection-value {
  font-size: 11px;
  color: var(--accent-primary);
}

.next-card-hint {
  display: flex;
  align-items: center;
  gap: 4px;
  margin-top: 8px;
  padding: 6px 10px;
  background: var(--bg-secondary);
  border-radius: 6px;
  font-size: 11px;
  color: var(--text-secondary);
}

.hint-icon {
  font-size: 14px;
}

.hint-text {
  flex: 1;
}

.next-card-rarity {
  font-weight: 600;
}

.character-detail {
  margin-top: 8px;
}

.detail-divider {
  height: 1px;
  background: var(--border-light);
  margin: 12px 0;
}

.detail-description {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.5;
  margin-bottom: 6px;
}

.detail-personality {
  font-size: 12px;
  color: var(--text-muted);
}
</style>

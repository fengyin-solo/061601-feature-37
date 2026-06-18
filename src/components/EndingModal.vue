<script setup lang="ts">
import { computed } from 'vue'
import { useGameStore } from '../stores/gameStore'
import gameConfig from '../config/gameConfig'
import { getRarityLabel } from '../utils/gameUtils'

const gameStore = useGameStore()

const ending = computed(() => gameStore.currentEnding)

const gradeColor = computed(() => {
  const grade = ending.value?.grade
  const colors: Record<string, string> = {
    S: 'linear-gradient(135deg, #f59e0b, #ef4444)',
    A: 'linear-gradient(135deg, #ec4899, #8b5cf6)',
    B: 'linear-gradient(135deg, #3b82f6, #06b6d4)',
    C: 'linear-gradient(135deg, #22c55e, #84cc16)',
    D: 'linear-gradient(135deg, #6b7280, #9ca3af)'
  }
  return colors[grade || 'D']
})

const collectedCards = computed(() => 
  gameConfig.cards.filter(c => gameStore.collectedCards.includes(c.id))
)

const collectedByRarity = computed(() => {
  const result: Record<string, number> = {
    common: 0,
    rare: 0,
    epic: 0,
    legendary: 0
  }
  collectedCards.value.forEach(card => {
    result[card.rarity] = (result[card.rarity] || 0) + 1
  })
  return result
})

const totalByRarity = computed(() => {
  const result: Record<string, number> = {
    common: 0,
    rare: 0,
    epic: 0,
    legendary: 0
  }
  gameConfig.cards.forEach(card => {
    result[card.rarity] = (result[card.rarity] || 0) + 1
  })
  return result
})

const highestAffinityCharacter = computed(() => {
  let maxAff = -Infinity
  let maxChar: typeof gameConfig.characters[0] | null = null
  gameStore.characters.forEach(char => {
    if (char.unlocked && char.affinity > maxAff) {
      maxAff = char.affinity
      const config = gameConfig.characters.find(c => c.id === char.id)
      if (config) maxChar = config
    }
  })
  return maxChar
})

const highestAffinity = computed(() => {
  return Math.max(...gameStore.characters.filter(c => c.unlocked).map(c => c.affinity), 0)
})

function restartGame() {
  gameStore.restartGame()
}
</script>

<template>
  <Teleport to="body">
    <Transition name="fade">
      <div v-if="gameStore.showEndingModal && ending" class="modal-overlay">
        <div class="modal-content ending-modal">
          <div class="ending-header">
            <div class="grade-badge" :style="{ background: gradeColor }">
              <span class="grade-letter">{{ ending.grade }}</span>
              <span class="grade-label">级结局</span>
            </div>
          </div>

          <h1 class="ending-title">{{ ending.title }}</h1>
          
          <p class="ending-description">{{ ending.description }}</p>

          <div v-if="highestAffinityCharacter" class="main-character">
            <div class="char-avatar-large">{{ highestAffinityCharacter.avatar }}</div>
            <div class="char-info">
              <span class="char-name">{{ highestAffinityCharacter.name }}</span>
              <span class="char-affinity">好感度 {{ highestAffinity }}</span>
            </div>
          </div>

          <div class="stats-section">
            <h3 class="stats-title">📊 游戏统计</h3>
            
            <div class="stats-grid">
              <div class="stat-card">
                <span class="stat-icon">🎴</span>
                <span class="stat-value">{{ gameStore.collectedCards.length }} / {{ gameConfig.cards.length }}</span>
                <span class="stat-label">卡牌收集</span>
              </div>
              <div class="stat-card">
                <span class="stat-icon">📈</span>
                <span class="stat-value">{{ gameStore.collectionRatePercent }}%</span>
                <span class="stat-label">完成度</span>
              </div>
              <div class="stat-card">
                <span class="stat-icon">📅</span>
                <span class="stat-value">{{ gameConfig.totalDays }} 天</span>
                <span class="stat-label">游戏时长</span>
              </div>
            </div>
          </div>

          <div class="rarity-stats">
            <h4>卡牌稀有度分布</h4>
            <div class="rarity-list">
              <div 
                v-for="(count, rarity) in totalByRarity" 
                :key="rarity"
                class="rarity-item"
              >
                <span class="rarity-name" :class="`rarity-${rarity}`">
                  {{ getRarityLabel(rarity) }}
                </span>
                <div class="rarity-bar">
                  <div 
                    class="rarity-fill"
                    :class="`fill-${rarity}`"
                    :style="{ width: `${count > 0 ? (collectedByRarity[rarity] || 0) / count * 100 : 0}%` }"
                  ></div>
                </div>
                <span class="rarity-count">{{ collectedByRarity[rarity] || 0 }}/{{ count }}</span>
              </div>
            </div>
          </div>

          <div class="ending-actions">
            <button class="btn btn-primary" @click="restartGame">
              🔄 重新开始
            </button>
          </div>

          <p class="ending-thanks">感谢游玩「恋爱物语」</p>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.ending-modal {
  padding: 36px;
  max-width: 500px;
  text-align: center;
  animation: fadeInUp 0.6s ease-out;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.ending-header {
  margin-bottom: 16px;
}

.grade-badge {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  padding: 16px 32px;
  border-radius: 16px;
  color: white;
}

.grade-letter {
  font-size: 48px;
  font-weight: 900;
  line-height: 1;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.grade-label {
  font-size: 14px;
  margin-top: 4px;
  opacity: 0.9;
}

.ending-title {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 12px;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.ending-description {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.8;
  margin-bottom: 24px;
  padding: 16px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-md);
}

.main-character {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 24px;
  padding: 16px;
  background: var(--accent-light);
  border-radius: var(--radius-md);
}

.char-avatar-large {
  font-size: 48px;
  width: 70px;
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--bg-secondary);
  border-radius: 50%;
}

.char-info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}

.char-name {
  font-size: 18px;
  font-weight: 600;
}

.char-affinity {
  font-size: 13px;
  color: var(--accent-primary);
  font-weight: 500;
}

.stats-section {
  margin-bottom: 20px;
}

.stats-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 12px;
  text-align: left;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.stat-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 14px 8px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-md);
}

.stat-icon {
  font-size: 24px;
}

.stat-value {
  font-size: 16px;
  font-weight: 700;
  color: var(--text-primary);
}

.stat-label {
  font-size: 11px;
  color: var(--text-secondary);
}

.rarity-stats {
  margin-bottom: 24px;
  text-align: left;
}

.rarity-stats h4 {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 10px;
  color: var(--text-secondary);
}

.rarity-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.rarity-item {
  display: flex;
  align-items: center;
  gap: 10px;
}

.rarity-name {
  width: 50px;
  font-size: 12px;
  font-weight: 500;
  flex-shrink: 0;
}

.rarity-common { color: #94a3b8; }
.rarity-rare { color: #3b82f6; }
.rarity-epic { color: #a855f7; }
.rarity-legendary { color: #f59e0b; }

.rarity-bar {
  flex: 1;
  height: 6px;
  background: var(--bg-secondary);
  border-radius: 3px;
  overflow: hidden;
}

.rarity-fill {
  height: 100%;
  border-radius: 3px;
  transition: width 0.5s ease;
}

.fill-common { background: #94a3b8; }
.fill-rare { background: #3b82f6; }
.fill-epic { background: #a855f7; }
.fill-legendary { background: #f59e0b; }

.rarity-count {
  width: 50px;
  font-size: 12px;
  color: var(--text-muted);
  text-align: right;
  flex-shrink: 0;
}

.ending-actions {
  margin-bottom: 16px;
}

.ending-actions .btn {
  width: 100%;
  padding: 14px;
  font-size: 16px;
  font-weight: 600;
}

.ending-thanks {
  font-size: 12px;
  color: var(--text-muted);
  font-style: italic;
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

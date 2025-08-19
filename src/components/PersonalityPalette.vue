<template>
  <div class="palette">
    <div v-if="!compact" class="palette__header">
      <h3 class="palette__title">Personality Palette</h3>
      <div class="palette__controls">
        <button 
          class="palette__toggle"
          :class="{ 'palette__toggle--active': viewMode === 'big5' }"
          @click="viewMode = 'big5'"
          aria-pressed="viewMode === 'big5'"
        >
          Big Five
        </button>
        <button 
          class="palette__toggle"
          :class="{ 'palette__toggle--active': viewMode === 'sales' }"
          @click="viewMode = 'sales'"
          aria-pressed="viewMode === 'sales'"
        >
          Sales Traits
        </button>
      </div>
    </div>
    
    <div class="palette__grid" @keydown="handleKeydown" tabindex="0" role="grid" aria-label="Personality traits">
      <div
        v-for="(trait, index) in sortedTraits"
        :key="trait.name"
        class="palette__tile"
        :style="{ backgroundColor: trait.color }"
        :tabindex="focusedIndex === index ? 0 : -1"
        role="gridcell"
        :aria-label="`${trait.name}: ${Math.round(trait.score * 100)}%`"
        @mouseenter="showTooltip(index)"
        @mouseleave="hideTooltip"
        @focus="showTooltip(index)"
        @blur="hideTooltip"
      >
        <div class="palette__tile-content">
          <div class="palette__tile-name">{{ trait.name }}</div>
          <div class="palette__tile-score">{{ Math.round(trait.score * 100) }}%</div>
        </div>
        
        <div 
          v-if="tooltipIndex === index && tooltip.visible"
          class="palette__tooltip"
          role="tooltip"
          :aria-describedby="`tooltip-${index}`"
        >
          <div class="palette__tooltip-score">
            {{ Math.round(trait.score * 100) }}% {{ trait.name }}
          </div>
          <div class="palette__tooltip-interpretation">
            {{ getInterpretation(trait) }}
          </div>
          <div class="palette__tooltip-source">
            {{ getRecentSource(trait) }}
          </div>
        </div>
      </div>
    </div>
    
    <div v-if="!compact" class="palette__legend">
      <p class="palette__legend-text">
        Colors represent trait intensity. Hover or focus on tiles for detailed insights and recent signals.
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

const props = defineProps({
  persona: {
    type: Object,
    required: true
  },
  compact: {
    type: Boolean,
    default: false
  }
})

const viewMode = ref('sales')
const tooltipIndex = ref(-1)
const focusedIndex = ref(0)
const tooltip = ref({ visible: false })

const sortedTraits = computed(() => {
  return [...props.persona.traits].sort((a, b) => b.score - a.score)
})

const interpretations = {
  'Analytical': {
    high: 'Prefers data-driven discussions and quantified outcomes',
    medium: 'Values evidence but open to intuitive approaches',
    low: 'Relies more on gut feeling and relationship-based decisions'
  },
  'Direct': {
    high: 'Appreciates clear, concise communication without fluff',
    medium: 'Values straightforward approach with some context',
    low: 'Prefers nuanced, relationship-focused conversations'
  },
  'Vision-led': {
    high: 'Motivated by big picture thinking and strategic outcomes',
    medium: 'Balances strategic vision with tactical execution',
    low: 'Focuses on immediate, practical results and processes'
  },
  'Risk-averse': {
    high: 'Prefers proven solutions and gradual implementation',
    medium: 'Open to innovation with proper validation',
    low: 'Embraces bold moves and disruptive approaches'
  },
  'Detail-oriented': {
    high: 'Values comprehensive information and thorough planning',
    medium: 'Appreciates key details while maintaining big picture view',
    low: 'Prefers high-level overviews and quick decisions'
  },
  'Empathic': {
    high: 'Values relationship building and team-focused outcomes',
    medium: 'Balances people and business considerations',
    low: 'Prioritizes business results over interpersonal dynamics'
  }
}

const getInterpretation = (trait) => {
  const level = trait.score > 0.7 ? 'high' : trait.score > 0.4 ? 'medium' : 'low'
  return interpretations[trait.name]?.[level] || 'Trait interpretation not available'
}

const getRecentSource = (trait) => {
  const recentSignal = props.persona.recentSignals[Math.floor(Math.random() * props.persona.recentSignals.length)]
  const types = {
    'podcast': 'Podcast quote',
    'press': 'Press coverage',
    'event': 'Event appearance'
  }
  const date = new Date(recentSignal.date).toLocaleDateString('en-US', { month: 'short', year: 'numeric' })
  return `${types[recentSignal.type]} • ${date}`
}

const showTooltip = async (index) => {
  tooltipIndex.value = index
  focusedIndex.value = index
  await nextTick()
  tooltip.value.visible = true
}

const hideTooltip = () => {
  tooltip.value.visible = false
  tooltipIndex.value = -1
}

const handleKeydown = (event) => {
  const traits = sortedTraits.value
  const currentIndex = focusedIndex.value
  let newIndex = currentIndex

  switch (event.key) {
    case 'ArrowRight':
      newIndex = (currentIndex + 1) % traits.length
      break
    case 'ArrowLeft':
      newIndex = currentIndex === 0 ? traits.length - 1 : currentIndex - 1
      break
    case 'ArrowDown':
      newIndex = Math.min(currentIndex + 3, traits.length - 1)
      break
    case 'ArrowUp':
      newIndex = Math.max(currentIndex - 3, 0)
      break
    case 'Enter':
    case ' ':
      event.preventDefault()
      showTooltip(currentIndex)
      return
    case 'Escape':
      hideTooltip()
      return
    default:
      return
  }

  event.preventDefault()
  focusedIndex.value = newIndex
  
  // Focus the new tile
  const tiles = event.currentTarget.querySelectorAll('.palette__tile')
  tiles[newIndex]?.focus()
}
</script>

<style scoped>
.palette {
  background: var(--panel);
  border-radius: var(--border-radius-lg);
  padding: 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
}

.palette__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.palette__title {
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
}

.palette__controls {
  display: flex;
  gap: 0.5rem;
}

.palette__toggle {
  padding: 0.5rem 1rem;
  border: 1px solid var(--muted);
  background: transparent;
  color: var(--muted);
  border-radius: var(--border-radius);
  font-family: inherit;
  font-size: 0.875rem;
  cursor: pointer;
  transition: var(--transition);
}

.palette__toggle:hover,
.palette__toggle:focus {
  border-color: var(--accent);
  color: var(--text);
}

.palette__toggle--active {
  background: var(--accent);
  color: var(--bg);
  border-color: var(--accent);
}

.palette__grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
  outline: none;
}

.palette__tile {
  position: relative;
  aspect-ratio: 1;
  border-radius: var(--border-radius);
  cursor: pointer;
  transition: var(--transition);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  outline: none;
  border: 2px solid transparent;
}

.palette__tile:hover,
.palette__tile:focus {
  transform: translateY(-2px) scale(1.02);
  border-color: var(--text);
  box-shadow: var(--shadow-lg);
}

.palette__tile-content {
  color: var(--bg);
  font-weight: 600;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.palette__tile-name {
  font-size: 0.875rem;
  margin-bottom: 0.25rem;
}

.palette__tile-score {
  font-size: 1.125rem;
}

.palette__tooltip {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: var(--bg);
  border: 1px solid rgba(166, 176, 187, 0.2);
  border-radius: var(--border-radius);
  padding: 1rem;
  min-width: 250px;
  z-index: 10;
  box-shadow: var(--shadow-lg);
  margin-top: 0.5rem;
  animation: tooltipFadeIn 0.2s ease;
}

@keyframes tooltipFadeIn {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(-4px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.palette__tooltip-score {
  font-weight: 600;
  color: var(--text);
  margin-bottom: 0.5rem;
}

.palette__tooltip-interpretation {
  font-size: 0.875rem;
  color: var(--muted);
  margin-bottom: 0.5rem;
  line-height: 1.4;
}

.palette__tooltip-source {
  font-size: 0.75rem;
  color: var(--accent);
  font-style: italic;
}

.palette__legend {
  text-align: center;
}

.palette__legend-text {
  font-size: 0.875rem;
  color: var(--muted);
  margin: 0;
}

@media (max-width: 768px) {
  .palette {
    padding: 1.5rem;
  }
  
  .palette__header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .palette__grid {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: 0.75rem;
  }
  
  .palette__tooltip {
    position: fixed;
    top: 50%;
    left: 1rem;
    right: 1rem;
    transform: translateY(-50%);
    min-width: auto;
  }
}

@media (prefers-reduced-motion: reduce) {
  .palette__tile {
    transition: none;
  }
  
  .palette__tile:hover,
  .palette__tile:focus {
    transform: none;
  }
  
  @keyframes tooltipFadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
}
</style>

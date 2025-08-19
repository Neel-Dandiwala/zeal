<template>
  <section class="hero section-large">
    <div class="container">
      <div class="hero__content">
        <div class="hero__text">
          <h1 class="hero__title">
            Real‑Time Global People Intelligence for Sales
          </h1>
          <p class="hero__subtitle">
            Zeal enriches prospects from worldwide sources and builds a living personality palette so you can write messages they'll actually respond to.
          </p>
          
          <div class="hero__actions">
            <a href="#demo" class="btn btn-primary">Start Free Trial</a>
            <a href="#demo" class="btn btn-secondary">Book Demo</a>
          </div>
          
          <p class="hero__trust">
            Integrates with Salesforce, HubSpot, Apollo, Clay.
          </p>
        </div>
        
            <div class="hero__visual">
      <PersonalityPaletteNebula 
        :traits="processedTraits" 
        :size="520"
        :animate="true"
      />
    </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed } from 'vue'
import PersonalityPaletteNebula from './PersonalityPaletteNebula.vue'

const props = defineProps({
  persona: {
    type: Object,
    required: true
  }
})

// Transform persona traits to match the nebula component's expected format
const processedTraits = computed(() => {
  const traits = props.persona.traits || []
  
  // Map traits to the five personality dimensions with brand colors
  const traitMap = {
    mind: { color: '#F24D2E', weight: 0.84 },      // Orange-red - Mind/Analytical
    energy: { color: '#6A73FF', weight: 0.78 },    // Periwinkle - Energy/Direct  
    nature: { color: '#0E57FF', weight: 0.66 },    // Royal blue - Nature/Vision-led
    tactics: { color: '#FFED69', weight: 0.41 },   // Yellow - Tactics/Risk-averse
    identity: { color: '#E53378', weight: 0.72 }   // Cerise - Identity/Detail-oriented
  }
  
  // Use actual trait data if available, otherwise use defaults
  if (traits.length >= 5) {
    traitMap.mind.weight = traits[0]?.score || 0.84
    traitMap.energy.weight = traits[1]?.score || 0.78  
    traitMap.nature.weight = traits[2]?.score || 0.66
    traitMap.tactics.weight = traits[3]?.score || 0.41
    traitMap.identity.weight = traits[4]?.score || 0.72
    
    // Use trait colors if provided
    if (traits[0]?.color) traitMap.mind.color = traits[0].color
    if (traits[1]?.color) traitMap.energy.color = traits[1].color
    if (traits[2]?.color) traitMap.nature.color = traits[2].color
    if (traits[3]?.color) traitMap.tactics.color = traits[3].color
    if (traits[4]?.color) traitMap.identity.color = traits[4].color
  }
  
  return traitMap
})
</script>

<style scoped>
.hero {
  margin-top: 80px;
  min-height: 80vh;
  display: flex;
  align-items: center;
}

.hero__content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 4rem;
  align-items: center;
}

.hero__title {
  font-size: 3.5rem;
  font-weight: 700;
  line-height: 1.1;
  margin-bottom: 1.5rem;
  background: linear-gradient(135deg, var(--text), var(--accent));
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.hero__subtitle {
  font-size: 1.25rem;
  color: var(--muted);
  margin-bottom: 2rem;
  line-height: 1.6;
}

.hero__actions {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.hero__trust {
  font-size: 0.875rem;
  color: var(--muted);
  font-style: italic;
}

.hero__visual {
  position: relative;
}

@media (max-width: 768px) {
  .hero {
    margin-top: 60px;
    min-height: auto;
  }
  
  .hero__content {
    grid-template-columns: 1fr;
    gap: 3rem;
    text-align: center;
  }
  
  .hero__title {
    font-size: 2.5rem;
  }
  
  .hero__subtitle {
    font-size: 1.125rem;
  }
  
  .hero__actions {
    justify-content: center;
    flex-wrap: wrap;
  }
}
</style>

<template>
  <section class="hero section">
    <div class="container">
      <div class="hero__content">
        <div class="hero__text">
          <h1 class="hero__title">
            Real‑Time Global People Intelligence for Sales
          </h1>
          <p class="hero__subtitle">
            Enrich prospects from worldwide sources and tailor outreach with a living personality palette.
          </p>
          
          <div class="hero__actions">
            <a href="#demo" class="btn btn-primary">Start Free Trial</a>
            <a href="#demo" class="btn btn-ghost">Book Demo</a>
          </div>
          
          <p class="hero__trust">
            Works with: <span class="trust-brands">Salesforce, HubSpot, Apollo, Clay, + more</span>
          </p>
        </div>
        
        <div class="hero__visual">
          <div class="palette-wrap">
            <PersonalityPaletteNebula 
              :traits="processedTraits" 
              :size="520"
              :animate="true"
            />
          </div>
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
  min-height: 100vh;
  display: flex;
  align-items: center;
  background: var(--bg);
}

.hero__content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-4xl);
  align-items: center;
}

.hero__text {
  max-width: 600px;
}

.hero__title {
  font-size: var(--text-3xl);
  font-weight: 700;
  line-height: 1.1;
  margin-bottom: var(--space-xl);
  color: var(--text);
}

.hero__subtitle {
  font-size: var(--text-lg);
  color: var(--muted);
  margin-bottom: var(--space-xl);
  line-height: 1.6;
  max-width: 55ch;
}

.hero__actions {
  display: flex;
  gap: var(--space-lg);
  margin-bottom: var(--space-xl);
  flex-wrap: wrap;
}

.hero__trust {
  font-size: var(--text-xs);
  color: var(--muted);
}

.trust-brands {
  color: var(--text);
  font-weight: 500;
}

.hero__visual {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: var(--space-3xl);
}

.palette-wrap {
  position: relative;
  width: min(46vw, 620px);
  aspect-ratio: 1 / 1;            /* ensures perfect square */
  border-radius: 50%;
  overflow: hidden;                /* hard circular clip */
  background: var(--bg);
  z-index: 0;
}

.palette-wrap::after {
  content: "";
  position: absolute;
  inset: 0;
  background: radial-gradient(
    ellipse at 50% 50%, 
    transparent 64%, 
    rgba(0, 0, 0, 0.18) 100%
  );
  pointer-events: none;
}

@media (max-width: 1024px) {
  .hero__content {
    gap: var(--space-3xl);
  }
  
  .hero__title {
    font-size: 2.75rem;
  }
}

@media (max-width: 768px) {
  .hero {
    margin-top: 60px;
    min-height: 90vh;
    padding: var(--space-xl) 0;
  }
  
  .hero__content {
    grid-template-columns: 1fr;
    gap: var(--space-3xl);
    text-align: center;
  }
  
  .hero__text {
    max-width: none;
    order: 1;
  }
  
  .hero__visual {
    order: 0;
    padding: var(--space-xl);
  }
  
  .hero__title {
    font-size: var(--text-2xl);
  }
  
  .hero__subtitle {
    font-size: var(--text-base);
    max-width: none;
  }
  
  .hero__actions {
    justify-content: center;
    gap: var(--space-md);
  }
  
  .palette-wrap {
    max-width: 400px;
  }
}

@media (max-width: 480px) {
  .hero {
    min-height: 80vh;
  }
  
  .hero__actions {
    flex-direction: column;
    align-items: center;
  }
  
  .btn {
    width: 100%;
    max-width: 280px;
  }
  
  .hero__visual {
    padding: var(--space-lg);
  }
  
  .palette-wrap {
    max-width: 320px;
  }
}
</style>

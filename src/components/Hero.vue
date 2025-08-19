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
          <PersonalityPalette :persona="persona" :compact="false" />
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import PersonalityPalette from './PersonalityPalette.vue'

defineProps({
  persona: {
    type: Object,
    required: true
  }
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

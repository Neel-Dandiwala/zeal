<template>
  <section id="integrations" class="integrations section">
    <div class="container">
      <div class="integrations__hero">
        <div class="integrations__content">
          <h2 class="integrations__title">Seamless Integrations</h2>
          <p class="integrations__subtitle">
            Drop‑in enrichment and brief generation on top of your existing stack
          </p>
          
          <!-- Auto-cycling logos strip -->
          <div class="integrations__logos-strip">
            <div class="logos-carousel">
              <div 
                v-for="integration in integrations" 
                :key="integration.name"
                class="logo-badge"
                :class="`logo-${integration.slug}`"
              >
                <span class="logo-text">{{ integration.name }}</span>
              </div>
            </div>
          </div>
          
          <div class="integrations__cta">
            <button class="btn btn-primary" @click="scrollToIntegrationsList">
              See it work with your tools
            </button>
          </div>
        </div>
        
        <div class="integrations__diagram">
          <IntegrationDiagram
            :center="diagramCenter"
            :inbound="diagramInbound"
            :outbound="diagramOutbound"
            :packet-speed="240"
            :loop-delay-ms="400"
          />
        </div>
      </div>
      
      <div class="integrations__features">
        <div class="grid grid-3">
          <div class="feature">
            <div class="feature__icon">
              <svg width="32" height="32" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M12 2L15.09 8.26L22 9L15.09 9.74L12 16L8.91 9.74L2 9L8.91 8.26L12 2Z" fill="currentColor"/>
              </svg>
            </div>
            <h3 class="feature__title">Real-time Sync</h3>
            <p class="feature__description">
              Automatically enriches new prospects as they enter your CRM pipeline.
            </p>
          </div>
          
          <div class="feature">
            <div class="feature__icon">
              <svg width="32" height="32" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M9 12L11 14L15 10M21 12C21 16.9706 16.9706 21 12 21C7.02944 21 3 16.9706 3 12C3 7.02944 7.02944 3 12 3C16.9706 3 21 7.02944 21 12Z" stroke="currentColor" stroke-width="2" fill="none"/>
              </svg>
            </div>
            <h3 class="feature__title">Zero Setup</h3>
            <p class="feature__description">
              Install in minutes with native integrations and pre-built field mappings.
            </p>
          </div>
          
          <div class="feature">
            <div class="feature__icon">
              <svg width="32" height="32" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M12 15L12 3M12 3L16 7M12 3L8 7M2 17L2 19C2 20.1046 2.89543 21 4 21L20 21C21.1046 21 22 20.1046 22 19L22 17" stroke="currentColor" stroke-width="2" fill="none"/>
              </svg>
            </div>
            <h3 class="feature__title">Data Ownership</h3>
            <p class="feature__description">
              All enriched data stays in your CRM. Export or migrate anytime.
            </p>
          </div>
        </div>
      </div>
      
      <div class="integrations__disclaimer">
        <p class="disclaimer-text">
          Logos are property of their respective owners. Zeal is an independent product.
        </p>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from 'vue'
import IntegrationDiagram from './IntegrationDiagram.vue'

const integrations = ref([
  { name: 'Salesforce', slug: 'salesforce' },
  { name: 'HubSpot', slug: 'hubspot' },
  { name: 'Apollo', slug: 'apollo' },
  { name: 'Clay', slug: 'clay' },
  { name: 'Outreach', slug: 'outreach' },
  { name: 'SalesLoft', slug: 'salesloft' }
])

// Diagram configuration
const diagramCenter = ref({
  id: 'zeal',
  label: 'Zeal',
  type: 'core',
  brandColor: '#2EF2E8',
  logo: { kind: 'mono', text: 'Z' }
})

const diagramInbound = ref([
  {
    id: 'hubspot',
    label: 'HubSpot',
    type: 'inbound',
    brandColor: '#FF7A59',
    logo: { kind: 'simpleIcon', slug: 'hubspot' }
  },
  {
    id: 'salesforce',
    label: 'Salesforce',
    type: 'inbound',
    brandColor: '#00A1E0',
    logo: { kind: 'simpleIcon', slug: 'salesforce' }
  },
  {
    id: 'apollo',
    label: 'Apollo',
    type: 'inbound',
    brandColor: '#6C5CE7',
    logo: { kind: 'mono', text: 'Ap' }
  },
  {
    id: 'clay',
    label: 'Clay',
    type: 'inbound',
    brandColor: '#00B894',
    logo: { kind: 'mono', text: 'Cl' }
  },
  {
    id: 'google',
    label: 'Google Search',
    type: 'inbound',
    brandColor: '#4285F4',
    logo: { kind: 'simpleIcon', slug: 'google' }
  },
  {
    id: 'bing',
    label: 'Bing',
    type: 'inbound',
    brandColor: '#0078D4',
    logo: { kind: 'simpleIcon', slug: 'bing' }
  }
])

const diagramOutbound = ref([
  {
    id: 'gmail',
    label: 'Gmail',
    type: 'outbound',
    brandColor: '#EA4335',
    logo: { kind: 'simpleIcon', slug: 'gmail' }
  },
  {
    id: 'slack',
    label: 'Slack',
    type: 'outbound',
    brandColor: '#4A154B',
    logo: { kind: 'simpleIcon', slug: 'slack' }
  },
  {
    id: 'discord',
    label: 'Discord',
    type: 'outbound',
    brandColor: '#5865F2',
    logo: { kind: 'simpleIcon', slug: 'discord' }
  },
  {
    id: 'whatsapp',
    label: 'WhatsApp',
    type: 'outbound',
    brandColor: '#25D366',
    logo: { kind: 'simpleIcon', slug: 'whatsapp' }
  },
  {
    id: 'linkedin',
    label: 'LinkedIn',
    type: 'outbound',
    brandColor: '#0A66C2',
    logo: { kind: 'simpleIcon', slug: 'linkedin' }
  },
  {
    id: 'web',
    label: 'Web',
    type: 'outbound',
    brandColor: '#6B7280',
    logo: { kind: 'simpleIcon', slug: 'globe' }
  }
])

const scrollToIntegrationsList = () => {
  const featuresSection = document.querySelector('.integrations__features')
  if (featuresSection) {
    featuresSection.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

  <style scoped>
.integrations {
  background: var(--bg);
}

.integrations__hero {
  display: flex;
  flex-direction: column;
  gap: 3rem;
  margin-bottom: 6rem;
}

.integrations__content {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
}

.integrations__title {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  line-height: 1.2;
}

.integrations__subtitle {
  font-size: 1.25rem;
  color: var(--muted);
  margin-bottom: 2rem;
  line-height: 1.6;
}

.integrations__logos-strip {
  margin-bottom: 2rem;
  overflow: hidden;
  padding: 1rem 0;
}

.logos-carousel {
  display: flex;
  gap: 1rem;
  animation: logoScroll 20s linear infinite;
}

.logo-badge {
  flex-shrink: 0;
  padding: 0.5rem 1rem;
  background: transparent;
  border: 1px solid rgba(166, 176, 187, 0.1);
  border-radius: var(--border-radius);
  font-size: 0.875rem;
  color: var(--text);
  white-space: nowrap;
  transition: var(--transition);
}

.logo-badge:hover {
  border-color: var(--accent);
  transform: translateY(-2px);
}

.integrations__cta {
  margin-top: 2rem;
}

.integrations__diagram {
  position: relative;
  width: 100%;
  background: transparent;
  border-radius: var(--border-radius-lg);
  border: 1px solid rgba(166, 176, 187, 0.08);
  padding: 2rem;
  backdrop-filter: blur(10px);
}

@keyframes logoScroll {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-50%);
  }
}

.logo-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 2rem;
  align-items: center;
  justify-items: center;
}

.logo-item {
  width: 100%;
  display: flex;
  justify-content: center;
}

.logo-placeholder {
  width: 160px;
  height: 80px;
  background: transparent;
  border: 2px solid rgba(166, 176, 187, 0.1);
  border-radius: var(--border-radius);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
  position: relative;
  overflow: hidden;
}

.logo-placeholder:hover {
  border-color: var(--accent);
  transform: translateY(-2px);
  box-shadow: var(--shadow);
}

.logo-text {
  font-weight: 600;
  color: var(--text);
  font-size: 1rem;
  text-align: center;
}

/* Branded logo styles */
.logo-salesforce {
  background: linear-gradient(135deg, #00a1e0, #0078d4);
}

.logo-salesforce .logo-text {
  color: white;
}

.logo-hubspot {
  background: linear-gradient(135deg, #ff7a59, #ff5722);
}

.logo-hubspot .logo-text {
  color: white;
}

.logo-apollo {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
}

.logo-apollo .logo-text {
  color: white;
}

.logo-clay {
  background: linear-gradient(135deg, #00b894, #00cec9);
}

.logo-clay .logo-text {
  color: white;
}

.logo-outreach {
  background: linear-gradient(135deg, #5b2c87, #9b59b6);
}

.logo-outreach .logo-text {
  color: white;
}

.logo-salesloft {
  background: linear-gradient(135deg, #2e86de, #54a0ff);
}

.logo-salesloft .logo-text {
  color: white;
}

.logo-more {
  background: var(--accent);
  border-color: var(--accent);
}

.logo-more .logo-text {
  color: var(--bg);
  font-weight: 500;
}

.integrations__features {
  margin-bottom: 3rem;
}

.feature {
  text-align: center;
  padding: 2rem 1rem;
}

.feature__icon {
  width: 64px;
  height: 64px;
  background: transparent;
  border: 1px solid rgba(166, 176, 187, 0.1);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1.5rem;
  color: var(--accent);
  transition: var(--transition);
}

.feature:hover .feature__icon {
  background: var(--accent);
  color: var(--bg);
  transform: scale(1.05);
}

.feature__title {
  font-size: 1.25rem;
  margin-bottom: 1rem;
  color: var(--text);
}

.feature__description {
  color: var(--muted);
  line-height: 1.6;
}

.integrations__disclaimer {
  text-align: center;
  padding-top: 2rem;
  border-top: 1px solid rgba(166, 176, 187, 0.1);
}

.disclaimer-text {
  font-size: 0.75rem;
  color: var(--muted);
  font-style: italic;
  margin: 0;
}

@media (max-width: 768px) {
  .integrations__hero {
    gap: 2rem;
  }
  
  .integrations__title {
    font-size: 2rem;
  }
  
  .integrations__subtitle {
    font-size: 1.125rem;
  }
  
  .integrations__diagram {
    padding: 1rem;
  }
  
  .logos-carousel {
    animation-duration: 15s;
  }
  
  .logo-grid {
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 1.5rem;
  }
  
  .logo-placeholder {
    width: 140px;
    height: 70px;
  }
  
  .logo-text {
    font-size: 0.875rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .logos-carousel {
    animation: none;
  }
  
  .logo-badge {
    transition: none;
  }
  
  .logo-badge:hover {
    transform: none;
  }
}

@media (prefers-reduced-motion: reduce) {
  .logo-placeholder:hover {
    transform: none;
  }
  
  .feature:hover .feature__icon {
    transform: none;
  }
}
</style>

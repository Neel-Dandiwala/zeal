<template>
  <section id="features" class="features section">
    <div class="container">
      <div class="text-center">
        <h2 class="features__title">Everything You Need</h2>
        <p class="features__subtitle">
          Comprehensive sales intelligence that integrates seamlessly with your workflow
        </p>
      </div>
      
      <div class="features__grid">
        <div 
          v-for="(feature, index) in features" 
          :key="feature.title"
          class="feature-card"
          :class="`feature-card--${index + 1}`"
        >
          <div class="feature-card__icon">
            <component :is="'div'" v-html="feature.icon"></component>
          </div>
          
          <h3 class="feature-card__title">{{ feature.title }}</h3>
          <p class="feature-card__description">{{ feature.description }}</p>
          
          <ul class="feature-card__benefits">
            <li v-for="benefit in feature.benefits" :key="benefit">
              {{ benefit }}
            </li>
          </ul>
          
          <div class="feature-card__highlight">
            <span class="highlight-badge">{{ feature.highlight }}</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from 'vue'

const features = ref([
  {
    title: 'Global Data Enrichment',
    description: 'Comprehensive prospect intelligence from worldwide sources including press, podcasts, events, and social media.',
    icon: `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M12 2C13.1 2 14 2.9 14 4C14 5.1 13.1 6 12 6C10.9 6 10 5.1 10 4C10 2.9 10.9 2 12 2ZM21 9V7L15 1L9 7V9C9 10 8 11 8 11S9 12 9 13V19H15V13C15 12 16 11 16 11S15 10 15 9V7L12 4L21 9Z" fill="currentColor"/>
    </svg>`,
    benefits: [
      'Real-time data from 15+ source types',
      'Multi-language content processing', 
      'Automated source reliability scoring',
      '99.2% data accuracy with verification'
    ],
    highlight: 'Live Updates'
  },
  {
    title: 'Personality Palette',
    description: 'AI-powered personality analysis that translates public signals into actionable communication insights.',
    icon: `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M12 2C13.1 2 14 2.9 14 4C14 5.1 13.1 6 12 6C10.9 6 10 5.1 10 4C10 2.9 10.9 2 12 2M21 9V7L15 1L9 7V9C9 10 8 11 8 11S9 12 9 13V19H15V13C15 12 16 11 16 11S15 10 15 9M9.5 11A1.5 1.5 0 1 1 11 12.5A1.5 1.5 0 0 1 9.5 11M14.5 11A1.5 1.5 0 1 1 16 12.5A1.5 1.5 0 0 1 14.5 11Z" fill="currentColor"/>
    </svg>`,
    benefits: [
      '6 core personality dimensions',
      'Communication style preferences',
      'Decision-making pattern analysis',
      'Custom sales trait mapping'
    ],
    highlight: 'AI-Powered'
  },
  {
    title: 'Actionable 2-minute Briefs',
    description: 'Concise, actionable prospect briefs with personality insights, recent activity, and conversation starters.',
    icon: `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M14 2H6C4.9 2 4 2.9 4 4V20C4 21.1 4.9 22 6 22H18C19.1 22 20 21.1 20 20V8L14 2M18 20H6V4H13V9H18V20Z" fill="currentColor"/>
      <path d="M8 12V14H16V12H8M8 16V18H13V16H8Z" fill="currentColor"/>
    </svg>`,
    benefits: [
      'Auto-generated meeting prep',
      'Conversation starter suggestions',
      'Recent activity highlights',
      'Personalized outreach templates'
    ],
    highlight: '2-Min Read'
  },
  {
    title: 'Source-Linked Evidence',
    description: 'Every insight backed by verifiable sources with direct links to original content for credibility.',
    icon: `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M10.59 13.41C11 13.8 11 14.4 10.59 14.81C10.2 15.2 9.6 15.2 9.19 14.81L7.78 13.4L6.37 14.81C5.96 15.2 5.36 15.2 4.95 14.81C4.54 14.4 4.54 13.8 4.95 13.39L6.36 12L4.95 10.61C4.54 10.2 4.54 9.6 4.95 9.19C5.36 8.78 5.96 8.78 6.37 9.19L7.78 10.6L9.19 9.19C9.6 8.78 10.2 8.78 10.61 9.19C11.02 9.6 11.02 10.2 10.61 10.61L9.2 12L10.59 13.41ZM13.41 9.19C13.8 8.78 14.4 8.78 14.81 9.19C15.2 9.6 15.2 10.2 14.81 10.61L13.4 12L14.81 13.39C15.2 13.8 15.2 14.4 14.81 14.81C14.4 15.2 13.8 15.2 13.39 14.81L12 13.4L10.61 14.81C10.2 15.2 9.6 15.2 9.19 14.81C8.78 14.4 8.78 13.8 9.19 13.39L10.6 12L9.19 10.61C8.78 10.2 8.78 9.6 9.19 9.19C9.6 8.78 10.2 8.78 10.61 9.19L12 10.6L13.41 9.19ZM17.41 6.34C18.78 7.71 19.55 9.53 19.55 11.48C19.55 13.43 18.78 15.25 17.41 16.62C16.04 17.99 14.22 18.76 12.27 18.76C10.32 18.76 8.5 17.99 7.13 16.62C5.76 15.25 4.99 13.43 4.99 11.48C4.99 9.53 5.76 7.71 7.13 6.34C8.5 4.97 10.32 4.2 12.27 4.2C14.22 4.2 16.04 4.97 17.41 6.34Z" fill="currentColor"/>
    </svg>`,
    benefits: [
      'Direct links to source content',
      'Publication date and context',
      'Source reliability scoring',
      'Fact-checking and verification'
    ],
    highlight: 'Verified'
  },
  {
    title: 'Role & Company Changes Alerts',
    description: 'Real-time notifications when prospects change roles, companies, or achieve career milestones.',
    icon: `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M10 20V14H14V20H19V12H22L12 3L2 12H5V20H10Z" fill="currentColor"/>
      <path d="M12 7.5C11.2 7.5 10.5 8.2 10.5 9S11.2 10.5 12 10.5 13.5 9.8 13.5 9 12.8 7.5 12 7.5Z" fill="currentColor"/>
    </svg>`,
    benefits: [
      'Job change notifications',
      'Company milestone tracking',
      'Promotion and role updates',
      'Strategic timing insights'
    ],
    highlight: 'Real-Time'
  },
  {
    title: 'Privacy & Controls',
    description: 'Enterprise-grade privacy controls with opt-out management and data compliance features.',
    icon: `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M12,1L3,5V11C3,16.55 6.84,21.74 12,23C17.16,21.74 21,16.55 21,11V5L12,1M12,7C13.4,7 14.8,8.6 14.8,10V11H16V16H8V11H9.2V10C9.2,8.6 10.6,7 12,7M12,8.2C11.2,8.2 10.5,8.7 10.5,10V11H13.5V10C13.5,8.7 12.8,8.2 12,8.2Z" fill="currentColor"/>
    </svg>`,
    benefits: [
      'GDPR and CCPA compliant',
      'Individual opt-out controls',
      'Data retention policies',
      'Audit trail and logging'
    ],
    highlight: 'Compliant'
  }
])
</script>

<style scoped>
.features__title {
  margin-bottom: 1rem;
}

.features__subtitle {
  font-size: 1.125rem;
  margin-bottom: 4rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.features__grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 2rem;
}

.feature-card {
  background: transparent;
  border-radius: var(--r-md);
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.04);
  transition: transform var(--dur-2) var(--e-out), 
              border-color var(--dur-1) var(--e-out);
  position: relative;
  overflow: hidden;
}

.feature-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: var(--accent);
  opacity: 0;
  transition: opacity var(--dur-1) var(--e-out);
}

.feature-card:hover {
  transform: translateY(-2px);
  border-color: rgba(255, 255, 255, 0.08);
}

.feature-card:hover::before {
  opacity: 1;
}

.feature-card__icon {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, var(--accent), var(--accent-2));
  border-radius: var(--border-radius);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1.5rem;
  color: var(--bg);
}

.feature-card__title {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1rem;
  color: var(--text);
}

.feature-card__description {
  color: var(--muted);
  margin-bottom: 1.5rem;
  line-height: 1.6;
}

.feature-card__benefits {
  list-style: none;
  padding: 0;
  margin: 0 0 1.5rem 0;
}

.feature-card__benefits li {
  padding: 0.5rem 0;
  color: var(--muted);
  font-size: 0.875rem;
  position: relative;
  padding-left: 1.5rem;
}

.feature-card__benefits li::before {
  content: '✓';
  position: absolute;
  left: 0;
  color: var(--accent);
  font-weight: bold;
}

.feature-card__highlight {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.highlight-badge {
  padding: 0.25rem 0.75rem;
  background: var(--accent);
  color: var(--bg);
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* Individual card accent colors */
.feature-card--2 .feature-card__icon {
  background: linear-gradient(135deg, var(--accent-2), #9b59b6);
}

.feature-card--2 .highlight-badge {
  background: var(--accent-2);
}

.feature-card--3 .feature-card__icon {
  background: linear-gradient(135deg, var(--warning), #e17055);
}

.feature-card--3 .highlight-badge {
  background: var(--warning);
}

.feature-card--4 .feature-card__icon {
  background: linear-gradient(135deg, #00b894, var(--accent));
}

.feature-card--4 .highlight-badge {
  background: #00b894;
}

.feature-card--5 .feature-card__icon {
  background: linear-gradient(135deg, #e84393, var(--accent-2));
}

.feature-card--5 .highlight-badge {
  background: #e84393;
}

.feature-card--6 .feature-card__icon {
  background: linear-gradient(135deg, #6c5ce7, var(--accent));
}

.feature-card--6 .highlight-badge {
  background: #6c5ce7;
}

@media (max-width: 768px) {
  .features__grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
  
  .feature-card {
    padding: 1.5rem;
  }
  
  .feature-card__icon {
    width: 64px;
    height: 64px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .feature-card:hover {
    transform: none;
  }
}
</style>

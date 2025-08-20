<template>
  <section class="testimonials section">
    <div class="container">
      <div class="text-center">
        <h2 class="testimonials__title">Trusted by Sales Leaders</h2>
        <p class="testimonials__subtitle">
          See how teams are transforming their outreach with personality-driven insights
        </p>
        <p class="testimonials__refresh">
          Last refreshed: {{ lastRefreshed }}
        </p>
      </div>
      
      <div class="testimonials__grid">
        <div 
          v-for="testimonial in testimonials" 
          :key="testimonial.name"
          class="testimonial-card"
        >
          <div class="testimonial-card__content">
            <blockquote class="testimonial-card__quote">
              "{{ testimonial.quote }}"
            </blockquote>
          </div>
          
          <div class="testimonial-card__author">
            <div class="testimonial-card__avatar">
              <img 
                :src="testimonial.avatar" 
                :alt="`${testimonial.name} avatar`"
                loading="lazy"
              />
            </div>
            <div class="testimonial-card__info">
              <div class="testimonial-card__name">{{ testimonial.name }}</div>
              <div class="testimonial-card__title">{{ testimonial.title }}</div>
              <div class="testimonial-card__company">{{ testimonial.company }}</div>
            </div>
          </div>
          
          <div class="testimonial-card__metrics">
            <div class="metric">
              <span class="metric__value">{{ testimonial.metrics.responseRate }}</span>
              <span class="metric__label">Response Rate Increase</span>
            </div>
            <div class="metric">
              <span class="metric__value">{{ testimonial.metrics.timeReduction }}</span>
              <span class="metric__label">Research Time Reduction</span>
            </div>
          </div>
        </div>
      </div>
      
      <div class="testimonials__stats">
        <div class="stats-grid">
          <div class="stat-item">
            <div class="stat-item__value">94%</div>
            <div class="stat-item__label">Customer Satisfaction</div>
          </div>
          <div class="stat-item">
            <div class="stat-item__value">3.2x</div>
            <div class="stat-item__label">Average Response Rate Lift</div>
          </div>
          <div class="stat-item">
            <div class="stat-item__value">68%</div>
            <div class="stat-item__label">Time Savings on Research</div>
          </div>
          <div class="stat-item">
            <div class="stat-item__value">500+</div>
            <div class="stat-item__label">Sales Teams Using Zeal</div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed } from 'vue'

const testimonials = ref([
  {
    name: 'Sarah Chen',
    title: 'VP Sales',
    company: 'TechFlow Dynamics',
    quote: 'Zeal completely transformed our outreach strategy. The personality insights help us connect with prospects on a deeper level, and our response rates have increased by 47%. The time we save on research lets us focus on actual selling.',
    avatar: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64"%3E%3Ccircle cx="32" cy="32" r="32" fill="%235bd7c2"/%3E%3Ctext x="32" y="38" text-anchor="middle" fill="white" font-family="Arial" font-size="20" font-weight="600"%3ESC%3C/text%3E%3C/svg%3E',
    metrics: {
      responseRate: '+47%',
      timeReduction: '71%'
    }
  },
  {
    name: 'Marcus Rodriguez',
    title: 'Head of Revenue Operations',
    company: 'CloudScale Solutions',
    quote: 'The AI-powered personality analysis is incredibly accurate. Our SDRs now approach each prospect with confidence, knowing exactly how to tailor their messaging. It\'s like having a psychology PhD on every call.',
    avatar: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64"%3E%3Ccircle cx="32" cy="32" r="32" fill="%237aa2ff"/%3E%3Ctext x="32" y="38" text-anchor="middle" fill="white" font-family="Arial" font-size="20" font-weight="600"%3EMR%3C/text%3E%3C/svg%3E',
    metrics: {
      responseRate: '+52%',
      timeReduction: '64%'
    }
  },
  {
    name: 'Emily Watson',
    title: 'Sales Director',
    company: 'InnovateLabs',
    quote: 'The real-time alerts for job changes and company updates are game-changing. We\'ve closed three major deals this quarter just by reaching out at the perfect moment when prospects changed roles.',
    avatar: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64"%3E%3Ccircle cx="32" cy="32" r="32" fill="%23f2b705"/%3E%3Ctext x="32" y="38" text-anchor="middle" fill="white" font-family="Arial" font-size="20" font-weight="600"%3EEW%3C/text%3E%3C/svg%3E',
    metrics: {
      responseRate: '+38%',
      timeReduction: '59%'
    }
  }
])

const lastRefreshed = computed(() => {
  const now = new Date()
  const options = { 
    month: 'short', 
    day: 'numeric', 
    year: 'numeric',
    hour: 'numeric',
    minute: '2-digit',
    hour12: true
  }
  return now.toLocaleDateString('en-US', options)
})
</script>

<style scoped>
.testimonials {
  background: transparent;
}

.testimonials__title {
  margin-bottom: 1rem;
}

.testimonials__subtitle {
  font-size: 1.125rem;
  margin-bottom: 1rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.testimonials__refresh {
  font-size: 0.875rem;
  color: var(--muted);
  font-style: italic;
  margin-bottom: 4rem;
}

.testimonials__grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 2rem;
  margin-bottom: 4rem;
}

.testimonial-card {
  background: var(--bg);
  border-radius: var(--border-radius-lg);
  padding: 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
  transition: var(--transition);
  position: relative;
}

.testimonial-card:hover {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: var(--shadow-lg);
}

.testimonial-card::before {
  content: '"';
  position: absolute;
  top: -10px;
  left: 2rem;
  font-size: 4rem;
  color: var(--accent);
  line-height: 1;
  font-family: serif;
}

.testimonial-card__content {
  margin-bottom: 2rem;
  padding-top: 1rem;
}

.testimonial-card__quote {
  font-size: 1.125rem;
  line-height: 1.6;
  color: var(--text);
  margin: 0;
  font-style: italic;
}

.testimonial-card__author {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid rgba(166, 176, 187, 0.1);
}

.testimonial-card__avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  overflow: hidden;
  flex-shrink: 0;
}

.testimonial-card__avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.testimonial-card__name {
  font-weight: 600;
  color: var(--text);
  margin-bottom: 0.25rem;
}

.testimonial-card__title {
  color: var(--accent);
  font-weight: 500;
  font-size: 0.875rem;
  margin-bottom: 0.25rem;
}

.testimonial-card__company {
  color: var(--muted);
  font-size: 0.875rem;
}

.testimonial-card__metrics {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.metric {
  text-align: center;
  padding: 1rem;
  background: rgba(91, 215, 194, 0.1);
  border-radius: var(--border-radius);
  border: 1px solid rgba(91, 215, 194, 0.2);
}

.metric__value {
  display: block;
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--accent);
  margin-bottom: 0.25rem;
}

.metric__label {
  font-size: 0.75rem;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.testimonials__stats {
  background: var(--bg);
  border-radius: var(--border-radius-lg);
  padding: 3rem 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 2rem;
}

.stat-item {
  text-align: center;
}

.stat-item__value {
  display: block;
  font-size: 3rem;
  font-weight: 700;
  background: linear-gradient(135deg, var(--accent), var(--accent-2));
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 0.5rem;
}

.stat-item__label {
  color: var(--muted);
  font-size: 0.875rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

@media (max-width: 768px) {
  .testimonials__grid {
    grid-template-columns: 1fr;
  }
  
  .testimonial-card {
    padding: 1.5rem;
  }
  
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1.5rem;
  }
  
  .stat-item__value {
    font-size: 2.5rem;
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
}

@media (prefers-reduced-motion: reduce) {
  .testimonial-card:hover {
    transform: none;
  }
}
</style>

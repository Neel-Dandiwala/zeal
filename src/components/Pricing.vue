<template>
  <section id="pricing" class="pricing section">
    <div class="container">
      <div class="text-center">
        <h2 class="pricing__title">Simple, Transparent Pricing</h2>
        <p class="pricing__subtitle">
          Choose the plan that fits your team size and data requirements
        </p>
      </div>
      
      <div class="pricing__toggle">
        <button 
          class="pricing__toggle-btn"
          :class="{ 'active': billingCycle === 'monthly' }"
          @click="billingCycle = 'monthly'"
        >
          Monthly
        </button>
        <button 
          class="pricing__toggle-btn"
          :class="{ 'active': billingCycle === 'annual' }"
          @click="billingCycle = 'annual'"
        >
          Annual <span class="pricing__discount">Save 20%</span>
        </button>
      </div>
      
      <div class="pricing__grid">
        <div 
          v-for="(plan, index) in plans" 
          :key="plan.name"
          class="pricing-card"
          :class="{ 
            'pricing-card--featured': plan.featured,
            [`pricing-card--${index + 1}`]: true 
          }"
        >
          <div class="pricing-card__header">
            <h3 class="pricing-card__name">{{ plan.name }}</h3>
            <div class="pricing-card__price">
              <span class="pricing-card__currency">$</span>
              <span class="pricing-card__amount">{{ getPrice(plan) }}</span>
              <span class="pricing-card__period">{{ getPeriod() }}</span>
            </div>
            <p class="pricing-card__description">{{ plan.description }}</p>
          </div>
          
          <div class="pricing-card__features">
            <ul class="features-list">
              <li v-for="feature in plan.features" :key="feature" class="features-list__item">
                {{ feature }}
              </li>
            </ul>
          </div>
          
          <div class="pricing-card__action">
            <button 
              v-if="plan.name !== 'Enterprise'"
              class="btn pricing-card__btn"
              :class="plan.featured ? 'btn-primary' : 'btn-secondary'"
            >
              Start {{ plan.name }} Plan
            </button>
            <button 
              v-else
              class="btn btn-secondary pricing-card__btn"
            >
              Talk to Sales
            </button>
          </div>
          
          <div v-if="plan.featured" class="pricing-card__badge">
            Most Popular
          </div>
        </div>
      </div>
      
      <div class="pricing__faq">
        <h3 class="pricing__faq-title">Pricing FAQ</h3>
        <div class="pricing__faq-grid">
          <div class="faq-item">
            <h4>What's included in data depth?</h4>
            <p>Data depth refers to the number of sources we scan and the historical timeframe. Higher tiers include more niche sources and longer lookback periods.</p>
          </div>
          <div class="faq-item">
            <h4>How often is data refreshed?</h4>
            <p>Starter: Weekly updates. Growth: Daily updates. Enterprise: Real-time updates with priority source monitoring.</p>
          </div>
          <div class="faq-item">
            <h4>Can I change plans anytime?</h4>
            <p>Yes, you can upgrade or downgrade at any time. Changes take effect immediately with prorated billing.</p>
          </div>
          <div class="faq-item">
            <h4>What about data compliance?</h4>
            <p>All plans include GDPR and CCPA compliance. Enterprise includes additional compliance certifications and audit capabilities.</p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed } from 'vue'

const billingCycle = ref('annual')

const plans = ref([
  {
    name: 'Starter',
    description: 'Perfect for individual sales reps and small teams getting started with personality-driven outreach.',
    monthlyPrice: 49,
    annualPrice: 39,
    featured: false,
    features: [
      '500 prospects per month',
      'Basic data sources (10 types)',
      'Weekly data refresh',
      '3 personality dimensions',
      'Email templates',
      'Salesforce integration',
      'Email support'
    ]
  },
  {
    name: 'Growth',
    description: 'Ideal for growing sales teams that need comprehensive intelligence and advanced personalization.',
    monthlyPrice: 149,
    annualPrice: 119,
    featured: true,
    features: [
      '2,500 prospects per month',
      'Premium data sources (15 types)',
      'Daily data refresh',
      '6 personality dimensions',
      'Multi-channel templates',
      'All CRM integrations',
      'Advanced personality insights',
      'Priority support',
      'Team collaboration'
    ]
  },
  {
    name: 'Enterprise',
    description: 'For large sales organizations requiring maximum data depth, custom integrations, and dedicated support.',
    monthlyPrice: 'Custom',
    annualPrice: 'Custom',
    featured: false,
    features: [
      'Unlimited prospects',
      'All data sources + custom',
      'Real-time data refresh',
      'Custom personality models',
      'White-label options',
      'Custom integrations',
      'Dedicated success manager',
      'SLA guarantees',
      'Advanced analytics',
      'Enterprise security'
    ]
  }
])

const getPrice = (plan) => {
  if (plan.monthlyPrice === 'Custom') return 'Custom'
  return billingCycle.value === 'monthly' ? plan.monthlyPrice : plan.annualPrice
}

const getPeriod = () => {
  return billingCycle.value === 'monthly' ? '/month' : '/month, billed annually'
}
</script>

<style scoped>
.pricing {
  background: linear-gradient(135deg, rgba(91, 215, 194, 0.05), rgba(122, 162, 255, 0.05));
}

.pricing__title {
  margin-bottom: 1rem;
}

.pricing__subtitle {
  font-size: 1.125rem;
  margin-bottom: 3rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.pricing__toggle {
  display: flex;
  justify-content: center;
  margin-bottom: 3rem;
  background: var(--panel);
  border-radius: var(--border-radius);
  padding: 0.5rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
  width: fit-content;
  margin-left: auto;
  margin-right: auto;
}

.pricing__toggle-btn {
  padding: 0.75rem 1.5rem;
  border: none;
  background: transparent;
  color: var(--muted);
  border-radius: var(--border-radius);
  font-family: inherit;
  font-weight: 500;
  cursor: pointer;
  transition: var(--transition);
  position: relative;
}

.pricing__toggle-btn.active {
  background: var(--accent);
  color: var(--bg);
}

.pricing__discount {
  font-size: 0.75rem;
  padding: 0.25rem 0.5rem;
  background: var(--warning);
  color: var(--bg);
  border-radius: 12px;
  margin-left: 0.5rem;
}

.pricing__grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 2rem;
  margin-bottom: 4rem;
}

.pricing-card {
  background: var(--panel);
  border-radius: var(--border-radius-lg);
  border: 1px solid rgba(166, 176, 187, 0.1);
  padding: 2rem;
  position: relative;
  transition: var(--transition);
  display: flex;
  flex-direction: column;
}

.pricing-card:hover {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: var(--shadow-lg);
}

.pricing-card--featured {
  border-color: var(--accent);
  transform: scale(1.05);
  background: linear-gradient(135deg, rgba(91, 215, 194, 0.05), rgba(122, 162, 255, 0.05));
}

.pricing-card--featured:hover {
  transform: scale(1.05) translateY(-4px);
}

.pricing-card__badge {
  position: absolute;
  top: -12px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--accent);
  color: var(--bg);
  padding: 0.5rem 1.5rem;
  border-radius: 12px;
  font-size: 0.875rem;
  font-weight: 600;
}

.pricing-card__header {
  text-align: center;
  margin-bottom: 2rem;
}

.pricing-card__name {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1rem;
  color: var(--text);
}

.pricing-card__price {
  margin-bottom: 1rem;
  display: flex;
  align-items: baseline;
  justify-content: center;
  gap: 0.25rem;
}

.pricing-card__currency {
  font-size: 1.5rem;
  color: var(--muted);
}

.pricing-card__amount {
  font-size: 3rem;
  font-weight: 700;
  color: var(--text);
}

.pricing-card__period {
  font-size: 1rem;
  color: var(--muted);
}

.pricing-card__description {
  color: var(--muted);
  line-height: 1.5;
}

.pricing-card__features {
  flex: 1;
  margin-bottom: 2rem;
}

.features-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.features-list__item {
  padding: 0.75rem 0;
  color: var(--muted);
  position: relative;
  padding-left: 1.5rem;
  border-bottom: 1px solid rgba(166, 176, 187, 0.1);
}

.features-list__item:last-child {
  border-bottom: none;
}

.features-list__item::before {
  content: '✓';
  position: absolute;
  left: 0;
  color: var(--accent);
  font-weight: bold;
}

.pricing-card__action {
  margin-top: auto;
}

.pricing-card__btn {
  width: 100%;
  justify-content: center;
}

.pricing__faq {
  text-align: center;
  padding-top: 3rem;
  border-top: 1px solid rgba(166, 176, 187, 0.1);
}

.pricing__faq-title {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  color: var(--text);
}

.pricing__faq-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  text-align: left;
}

.faq-item h4 {
  font-size: 1.125rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: var(--text);
}

.faq-item p {
  color: var(--muted);
  line-height: 1.5;
  margin: 0;
}

@media (max-width: 768px) {
  .pricing__grid {
    grid-template-columns: 1fr;
  }
  
  .pricing-card--featured {
    transform: none;
  }
  
  .pricing-card--featured:hover {
    transform: translateY(-4px);
  }
  
  .pricing__toggle {
    width: 100%;
  }
  
  .pricing__faq-grid {
    grid-template-columns: 1fr;
  }
}

@media (prefers-reduced-motion: reduce) {
  .pricing-card:hover,
  .pricing-card--featured:hover {
    transform: none;
  }
}
</style>

<template>
  <section id="faq" class="faq section">
    <div class="container">
      <div class="text-center">
        <h2 class="faq__title">Frequently Asked Questions</h2>
        <p class="faq__subtitle">
          Everything you need to know about Zeal's data sources, accuracy, and implementation
        </p>
      </div>
      
      <div class="faq__list">
        <div 
          v-for="(item, index) in faqItems" 
          :key="index"
          class="faq-item"
          :class="{ 'faq-item--open': openItems.includes(index) }"
        >
          <button 
            class="faq-item__trigger"
            @click="toggleItem(index)"
            :aria-expanded="openItems.includes(index)"
            :aria-controls="`faq-content-${index}`"
          >
            <span class="faq-item__question">{{ item.question }}</span>
            <span class="faq-item__icon" aria-hidden="true">
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M6 9L12 15L18 9" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </span>
          </button>
          
          <div 
            :id="`faq-content-${index}`"
            class="faq-item__content"
            :aria-hidden="!openItems.includes(index)"
          >
            <div class="faq-item__answer">
              <p v-for="(paragraph, pIndex) in item.answer" :key="pIndex">
                {{ paragraph }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from 'vue'

const openItems = ref([0]) // First item open by default

const faqItems = ref([
  {
    question: 'What data sources does Zeal use?',
    answer: [
      'Zeal aggregates data from 15+ global sources including press releases, podcast transcripts, conference speaker lists, company announcements, social media, alumni directories, board/foundation listings, and local-language media.',
      'We prioritize publicly available information and respect privacy controls. All sources are verified for reliability and recency.'
    ]
  },
  {
    question: 'How do you handle consent and privacy controls?',
    answer: [
      'Zeal is GDPR and CCPA compliant. We only use publicly available information and provide opt-out mechanisms for individuals who prefer not to be included.',
      'Enterprise customers get additional privacy controls including data retention policies, audit trails, and custom compliance settings.'
    ]
  },
  {
    question: 'How accurate is the personality analysis?',
    answer: [
      'Our AI models achieve 84% accuracy when validated against professional personality assessments. The system improves continuously by learning from successful outreach patterns.',
      'We provide confidence scores for each trait and always link back to source material so you can verify insights yourself.'
    ]
  },
  {
    question: 'How quickly can I set up Zeal with my CRM?',
    answer: [
      'Most integrations take 5-10 minutes using our pre-built connectors for Salesforce, HubSpot, Apollo, and other major CRMs.',
      'Our field mapping is pre-configured for common CRM setups, but you can customize it to match your specific data structure.'
    ]
  },
  {
    question: 'What security measures protect my data?',
    answer: [
      'All data is encrypted in transit and at rest. We maintain SOC 2 Type II compliance and undergo regular security audits.',
      'Your CRM data stays in your system - we only enrich it with additional insights. You can export or delete all Zeal-generated data at any time.'
    ]
  },
  {
    question: 'Can I customize the personality traits and analysis?',
    answer: [
      'Yes! While we provide standard Big Five and sales-specific traits, Enterprise customers can define custom personality dimensions relevant to their industry.',
      'You can also adjust the weighting of different data sources based on what matters most for your sales process.'
    ]
  },
  {
    question: 'How often is prospect data updated?',
    answer: [
      'Update frequency depends on your plan: Starter (weekly), Growth (daily), Enterprise (real-time).',
      'We also provide instant alerts for major changes like job switches, company news, or significant public appearances.'
    ]
  },
  {
    question: 'What happens if I need to cancel or change plans?',
    answer: [
      'You can upgrade, downgrade, or cancel at any time. Changes take effect immediately with prorated billing.',
      'All enriched data in your CRM remains yours permanently. We provide export tools to ensure you never lose access to insights generated during your subscription.'
    ]
  }
])

const toggleItem = (index) => {
  const isOpen = openItems.value.includes(index)
  if (isOpen) {
    openItems.value = openItems.value.filter(i => i !== index)
  } else {
    openItems.value.push(index)
  }
}
</script>

<style scoped>
.faq {
  background: var(--bg);
}

.faq__title {
  margin-bottom: 1rem;
}

.faq__subtitle {
  font-size: 1.125rem;
  margin-bottom: 4rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.faq__list {
  max-width: 800px;
  margin: 0 auto;
}

.faq-item {
  background: transparent;
  border-radius: var(--border-radius);
  border: 1px solid rgba(255, 255, 255, 0.04);
  margin-bottom: 1rem;
  overflow: hidden;
  transition: var(--transition);
}

.faq-item:hover {
  border-color: var(--accent);
}

.faq-item--open {
  border-color: var(--accent);
  box-shadow: var(--shadow);
}

.faq-item__trigger {
  width: 100%;
  padding: 1.5rem;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  font-family: inherit;
  transition: var(--transition);
}

.faq-item__trigger:hover {
  background: rgba(91, 215, 194, 0.05);
}

.faq-item__trigger:focus {
  outline: 2px solid var(--accent);
  outline-offset: -2px;
}

.faq-item__question {
  font-size: 1.125rem;
  font-weight: 600;
  color: var(--text);
  line-height: 1.4;
}

.faq-item__icon {
  color: var(--accent);
  flex-shrink: 0;
  transition: var(--transition);
}

.faq-item--open .faq-item__icon {
  transform: rotate(180deg);
}

.faq-item__content {
  max-height: 0;
  overflow: hidden;
  transition: var(--transition);
}

.faq-item--open .faq-item__content {
  max-height: 500px;
}

.faq-item__answer {
  padding: 0 1.5rem 1.5rem;
}

.faq-item__answer p {
  color: var(--muted);
  line-height: 1.6;
  margin-bottom: 1rem;
}

.faq-item__answer p:last-child {
  margin-bottom: 0;
}

@media (max-width: 768px) {
  .faq-item__trigger {
    padding: 1.25rem;
  }
  
  .faq-item__question {
    font-size: 1rem;
  }
  
  .faq-item__answer {
    padding: 0 1.25rem 1.25rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .faq-item__content {
    transition: none;
  }
  
  .faq-item--open .faq-item__content {
    max-height: none;
  }
  
  .faq-item__icon {
    transition: none;
  }
}
</style>

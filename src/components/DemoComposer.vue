<template>
  <section id="demo" class="demo section">
    <div class="container">
      <div class="text-center">
        <h2 class="demo__title">Live Demo: Generate Outreach</h2>
        <p class="demo__subtitle">
          See how Zeal transforms personality insights into compelling, personalized outreach
        </p>
      </div>
      
      <div class="demo__content">
        <div class="demo__persona">
          <div class="persona-card">
            <div class="persona-card__header">
              <h3 class="persona-card__name">{{ persona.name }}</h3>
              <p class="persona-card__role">{{ persona.role }}</p>
              <p class="persona-card__company">{{ persona.company }}</p>
              <p class="persona-card__location">{{ persona.location }}</p>
            </div>
            
            <div class="persona-card__traits">
              <h4>Top Traits</h4>
              <div class="traits-chips">
                <span 
                  v-for="trait in topTraits" 
                  :key="trait.name"
                  class="trait-chip"
                  :style="{ backgroundColor: trait.color }"
                >
                  {{ trait.name }} ({{ Math.round(trait.score * 100) }}%)
                </span>
              </div>
            </div>
            
            <div class="persona-card__signals">
              <h4>Recent Signals</h4>
              <div class="signals-list">
                <div 
                  v-for="signal in persona.recentSignals" 
                  :key="signal.date"
                  class="signal-item"
                >
                  <div class="signal-type">{{ formatSignalType(signal.type) }}</div>
                  <div class="signal-title">{{ signal.title }}</div>
                  <div class="signal-note">{{ signal.note }}</div>
                  <div class="signal-date">{{ formatDate(signal.date) }}</div>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <div class="demo__composer">
          <div class="composer">
            <div class="composer__header">
              <div class="composer__tabs">
                <button 
                  v-for="channel in channels"
                  :key="channel"
                  class="composer__tab"
                  :class="{ 'composer__tab--active': activeChannel === channel }"
                  @click="activeChannel = channel"
                >
                  {{ channel }}
                </button>
              </div>
              
              <div class="composer__tone">
                <label for="tone-select" class="composer__tone-label">Tone:</label>
                <select 
                  id="tone-select"
                  v-model="selectedTone" 
                  class="composer__tone-select"
                  @change="generateSample"
                >
                  <option value="concise">Concise</option>
                  <option value="analytical">Analytical</option>
                  <option value="friendly">Friendly</option>
                </select>
              </div>
            </div>
            
            <div class="composer__content">
              <textarea 
                v-model="generatedContent"
                class="composer__textarea"
                :placeholder="`Generated ${activeChannel.toLowerCase()} will appear here...`"
                readonly
              ></textarea>
              
              <div class="composer__actions">
                <button 
                  class="btn btn-primary composer__generate"
                  @click="generateSample"
                  :disabled="isGenerating"
                >
                  <span v-if="isGenerating">Generating...</span>
                  <span v-else>Generate {{ activeChannel }}</span>
                </button>
                
                <button 
                  class="btn btn-secondary"
                  @click="copySample"
                  :disabled="!generatedContent"
                >
                  Copy Text
                </button>
              </div>
            </div>
            
            <div v-if="generatedContent" class="composer__insights">
              <h4>Personalization Insights</h4>
              <ul class="insights-list">
                <li>Emphasized data-driven approach (Analytical: 84%)</li>
                <li>Used direct, concise language (Direct: 78%)</li>
                <li>Referenced recent RevOps podcast appearance</li>
                <li>Focused on enterprise expansion metrics</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  persona: {
    type: Object,
    required: true
  }
})

const channels = ['Email', 'LinkedIn DM', 'Meeting Notes']
const activeChannel = ref('Email')
const selectedTone = ref('concise')
const generatedContent = ref('')
const isGenerating = ref(false)

const topTraits = computed(() => {
  return props.persona.traits
    .sort((a, b) => b.score - a.score)
    .slice(0, 3)
})

const formatSignalType = (type) => {
  const types = {
    'podcast': 'Podcast',
    'press': 'Press',
    'event': 'Event'
  }
  return types[type] || type
}

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('en-US', { 
    month: 'short', 
    day: 'numeric',
    year: 'numeric'
  })
}

const generateSample = async () => {
  isGenerating.value = true
  
  // Simulate API delay
  await new Promise(resolve => setTimeout(resolve, 1500))
  
  generatedContent.value = generateCopy(props.persona, activeChannel.value, selectedTone.value)
  isGenerating.value = false
}

const generateCopy = (persona, channel, tone) => {
  const templates = {
    'Email': {
      'concise': `Subject: Quick question about Finch's enterprise expansion

Hi ${persona.name},

Saw your insights on RevOps Unpacked about data-driven experimentation. Given Finch's Series B extension focus on enterprise expansion, I'm curious about your metrics framework for scaling sales operations.

We help VP Sales leaders like you at growth-stage robotics companies reduce prospect research time by 75% with AI-powered personality profiling. 

Worth a 15-minute conversation?

Best,
[Your name]`,
      
      'analytical': `Subject: Data-driven approach to enterprise sales scaling - Finch Robotics

Hi ${persona.name},

Your recent commentary on RevOps Unpacked about emphasizing data-driven experimentation resonated with our approach to sales intelligence. 

Given Finch's Series B extension and focus on enterprise expansion, I imagine you're evaluating tools that can quantify prospect engagement patterns and optimize outreach effectiveness.

Our platform analyzes 15+ personality dimensions from public data sources, generating measurable insights that have helped similar VP Sales leaders achieve:
- 34% higher response rates through personalized messaging
- 67% reduction in prospect research time  
- 23% improvement in qualification-to-close rates

The data shows this approach works particularly well for technical sales teams selling complex solutions to enterprise buyers.

Would you be interested in seeing the specific metrics and methodology? Happy to share a brief analysis of your current prospect base.

Best regards,
[Your name]`,
      
      'friendly': `Subject: Loved your RevOps Unpacked insights! 

Hi ${persona.name},

Hope you're doing well! I caught your appearance on RevOps Unpacked and really appreciated your thoughts on data-driven experimentation. It's clear you're thinking strategically about scaling operations at Finch.

I've been working with VP Sales leaders at growth-stage companies who are tackling similar challenges around enterprise expansion. Many have found that understanding prospect personalities and communication preferences can be a game-changer for outreach effectiveness.

I'd love to share some insights that might be relevant to what you're building at Finch. No pitch - just thought the approach might interest you given your focus on data-driven strategies.

Would you be up for a quick chat?

Cheers,
[Your name]`
    },
    
    'LinkedIn DM': {
      'concise': `Hi ${persona.name}, saw your RevOps Unpacked appearance - great insights on data-driven experimentation. Given Finch's enterprise focus, curious about your approach to sales intelligence tools. Worth connecting?`,
      
      'analytical': `Hi ${persona.name}, your RevOps Unpacked insights on data-driven experimentation align with our approach to sales intelligence. Our platform helps enterprise sales teams like Finch's achieve 34% higher response rates through personality-based prospect profiling. Would you be interested in seeing the methodology and metrics?`,
      
      'friendly': `Hey ${persona.name}! Really enjoyed your thoughts on RevOps Unpacked. As someone building enterprise sales operations, thought you might find our approach to prospect intelligence interesting. Would love to connect and share some insights!`
    },
    
    'Meeting Notes': {
      'concise': `**Pre-meeting Brief: ${persona.name}**

**Key Personality Traits:**
- Highly analytical (84%) - values data-driven discussions
- Direct communicator (78%) - prefers concise presentations
- Vision-led (66%) - motivated by strategic outcomes

**Recent Context:**
- RevOps Unpacked podcast: emphasized experimentation
- Finch Series B extension: focused on enterprise expansion
- SaaStr panel: prefers metric-led pitches

**Recommended Approach:**
- Lead with quantified outcomes and ROI metrics
- Keep presentation concise and data-heavy
- Focus on enterprise scaling challenges
- Reference recent podcast insights to build rapport`,
      
      'analytical': `**Comprehensive Meeting Brief: ${persona.name}**

**Personality Profile:**
- Analytical: 84% - Requires quantified evidence and ROI justification
- Direct: 78% - Values efficiency, minimal small talk
- Vision-led: 66% - Interested in strategic transformation
- Detail-oriented: 72% - Appreciates thorough methodology explanation
- Risk-averse: 41% - Open to innovation with proper validation

**Recent Intelligence:**
1. RevOps Unpacked (July 22) - Emphasized data-driven experimentation
2. Finch Series B Extension (June 30) - Focus on enterprise expansion  
3. SaaStr Panel (June 10) - Prefers concise, metric-led pitches

**Strategic Context:**
- Company: Growth-stage robotics, enterprise pivot
- Role: Scaling North American sales operations
- Challenges: Likely include enterprise sales cycle complexity, team scaling

**Meeting Strategy:**
- Open with relevant industry benchmarks
- Present ROI framework within first 5 minutes
- Demonstrate measurable impact on similar companies
- Prepare detailed methodology backup slides
- Reference RevOps podcast to establish credibility`,
      
      'friendly': `**Meeting Prep: ${persona.name}**

**Personal Context:**
- Recently featured on RevOps Unpacked - great insights!
- Leading North American expansion at Finch Robotics
- Based in San Francisco, focused on enterprise growth

**Communication Style:**
- Analytical but approachable
- Values relationship building alongside business results
- Appreciates authentic connections

**Conversation Starters:**
- Ask about RevOps podcast experience
- Discuss enterprise sales challenges in robotics
- Share relevant insights from similar companies

**Key Points to Cover:**
- How we've helped similar VP Sales leaders
- Specific success stories in enterprise scaling
- Collaborative approach to implementation

**Tone for Meeting:**
- Professional but warm
- Consultative rather than salesy
- Focus on mutual value and learning`
    }
  }
  
  return templates[channel]?.[tone] || 'Sample content not available for this combination.'
}

const copySample = async () => {
  try {
    await navigator.clipboard.writeText(generatedContent.value)
  } catch (err) {
    // Fallback for older browsers
    const textArea = document.createElement('textarea')
    textArea.value = generatedContent.value
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
  }
}

onMounted(() => {
  generateSample()
})
</script>

<style scoped>
.demo {
  background: var(--panel);
}

.demo__title {
  margin-bottom: 1rem;
}

.demo__subtitle {
  font-size: 1.125rem;
  margin-bottom: 4rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.demo__content {
  display: grid;
  grid-template-columns: 1fr 1.5fr;
  gap: 3rem;
  align-items: start;
}

.persona-card {
  background: var(--bg);
  border-radius: var(--border-radius-lg);
  padding: 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
  position: sticky;
  top: 100px;
}

.persona-card__header {
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid rgba(166, 176, 187, 0.1);
}

.persona-card__name {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: var(--text);
}

.persona-card__role {
  font-weight: 500;
  color: var(--accent);
  margin-bottom: 0.25rem;
}

.persona-card__company,
.persona-card__location {
  color: var(--muted);
  font-size: 0.875rem;
  margin-bottom: 0.25rem;
}

.persona-card__traits h4,
.persona-card__signals h4 {
  font-size: 1rem;
  margin-bottom: 1rem;
  color: var(--text);
}

.persona-card__traits {
  margin-bottom: 2rem;
}

.traits-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.trait-chip {
  padding: 0.375rem 0.75rem;
  border-radius: var(--border-radius);
  font-size: 0.75rem;
  font-weight: 500;
  color: var(--bg);
  white-space: nowrap;
}

.signals-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.signal-item {
  padding: 1rem;
  background: var(--panel);
  border-radius: var(--border-radius);
  border: 1px solid rgba(166, 176, 187, 0.1);
}

.signal-type {
  font-size: 0.75rem;
  color: var(--accent);
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.signal-title {
  font-weight: 500;
  color: var(--text);
  margin: 0.25rem 0;
}

.signal-note {
  font-size: 0.875rem;
  color: var(--muted);
  margin-bottom: 0.5rem;
}

.signal-date {
  font-size: 0.75rem;
  color: var(--muted);
}

.composer {
  background: var(--bg);
  border-radius: var(--border-radius-lg);
  border: 1px solid rgba(166, 176, 187, 0.1);
  overflow: hidden;
}

.composer__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid rgba(166, 176, 187, 0.1);
  background: rgba(166, 176, 187, 0.05);
}

.composer__tabs {
  display: flex;
  gap: 0.5rem;
}

.composer__tab {
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

.composer__tab:hover,
.composer__tab:focus {
  border-color: var(--accent);
  color: var(--text);
}

.composer__tab--active {
  background: var(--accent);
  color: var(--bg);
  border-color: var(--accent);
}

.composer__tone {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.composer__tone-label {
  font-size: 0.875rem;
  color: var(--muted);
}

.composer__tone-select {
  padding: 0.5rem;
  border: 1px solid var(--muted);
  background: var(--panel);
  color: var(--text);
  border-radius: var(--border-radius);
  font-family: inherit;
  font-size: 0.875rem;
}

.composer__content {
  padding: 1.5rem;
}

.composer__textarea {
  width: 100%;
  min-height: 300px;
  padding: 1rem;
  border: 1px solid rgba(166, 176, 187, 0.2);
  background: var(--panel);
  color: var(--text);
  border-radius: var(--border-radius);
  font-family: var(--font-mono);
  font-size: 0.875rem;
  line-height: 1.5;
  resize: vertical;
  margin-bottom: 1rem;
}

.composer__textarea:focus {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
  border-color: var(--accent);
}

.composer__actions {
  display: flex;
  gap: 1rem;
}

.composer__generate:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.composer__insights {
  padding: 1.5rem;
  border-top: 1px solid rgba(166, 176, 187, 0.1);
  background: rgba(91, 215, 194, 0.05);
}

.composer__insights h4 {
  font-size: 1rem;
  margin-bottom: 1rem;
  color: var(--text);
}

.insights-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.insights-list li {
  padding: 0.5rem 0;
  color: var(--muted);
  font-size: 0.875rem;
  position: relative;
  padding-left: 1.5rem;
}

.insights-list li::before {
  content: '•';
  color: var(--accent);
  position: absolute;
  left: 0;
  font-weight: bold;
}

@media (max-width: 768px) {
  .demo__content {
    grid-template-columns: 1fr;
    gap: 2rem;
  }
  
  .persona-card {
    position: static;
  }
  
  .composer__header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .composer__tabs {
    justify-content: center;
  }
  
  .composer__actions {
    flex-direction: column;
  }
}
</style>

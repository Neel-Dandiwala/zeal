<template>
  <section id="how-it-works" class="how-it-works section">
    <div class="container">
      <div class="text-center">
        <h2 class="how-it-works__title">How It Works</h2>
        <p class="how-it-works__subtitle">
          Three steps to transform your outreach with global intelligence
        </p>
      </div>
      
      <div class="how-it-works__steps">
        <div class="step">
          <div class="step__icon">
            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M12 2L13.09 8.26L20 9L13.09 9.74L12 16L10.91 9.74L4 9L10.91 8.26L12 2Z" fill="currentColor"/>
              <path d="M12 17L13.09 20.26L16 21L13.09 21.74L12 25L10.91 21.74L8 21L10.91 20.26L12 17Z" fill="currentColor"/>
            </svg>
          </div>
          <h3 class="step__title">Ingest</h3>
          <p class="step__description">
            Global web + local language sources, podcasts, news, conference agendas, 
            alumni pages, boards/foundations, social media.
          </p>
        </div>
        
        <div class="step">
          <div class="step__icon">
            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M9.5 3A6.5 6.5 0 0 1 16 9.5C16 11.11 15.41 12.59 14.44 13.73L14.71 14H15.5L20.5 19L19 20.5L14 15.5V14.71L13.73 14.44C12.59 15.41 11.11 16 9.5 16A6.5 6.5 0 0 1 3 9.5A6.5 6.5 0 0 1 9.5 3M9.5 5C7 5 5 7 5 9.5S7 14 9.5 14 14 12 14 9.5 12 5 9.5 5Z" fill="currentColor"/>
              <circle cx="9.5" cy="9.5" r="2.5" fill="currentColor"/>
            </svg>
          </div>
          <h3 class="step__title">Enrich</h3>
          <p class="step__description">
            Entity resolution, dedupe, recency scoring, reliability weighting. 
            Build comprehensive profiles with confidence indicators.
          </p>
        </div>
        
        <div class="step">
          <div class="step__icon">
            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M14,2H6A2,2 0 0,0 4,4V20A2,2 0 0,0 6,22H18A2,2 0 0,0 20,20V8L14,2M18,20H6V4H13V9H18V20Z" fill="currentColor"/>
              <path d="M8,12V14H16V12H8M8,16V18H13V16H8Z" fill="currentColor"/>
            </svg>
          </div>
          <h3 class="step__title">Compose</h3>
          <p class="step__description">
            Generate brief + outreach tailored to persona and history. 
            Adapt tone and approach based on personality insights.
          </p>
        </div>
      </div>
      
      <div class="how-it-works__diagram">
        <svg class="diagram" viewBox="0 0 800 200" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <linearGradient id="flowGradient" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%" style="stop-color:var(--accent-2);stop-opacity:1" />
              <stop offset="100%" style="stop-color:var(--accent);stop-opacity:1" />
            </linearGradient>
          </defs>
          
          <!-- Sources -->
          <g class="diagram__section">
            <rect x="20" y="40" width="120" height="120" rx="8" fill="var(--panel)" stroke="var(--muted)"/>
            <text x="80" y="60" text-anchor="middle" fill="var(--text)" font-size="14" font-weight="600">Sources</text>
            <text x="80" y="80" text-anchor="middle" fill="var(--muted)" font-size="10">Global Web</text>
            <text x="80" y="95" text-anchor="middle" fill="var(--muted)" font-size="10">Podcasts</text>
            <text x="80" y="110" text-anchor="middle" fill="var(--muted)" font-size="10">News</text>
            <text x="80" y="125" text-anchor="middle" fill="var(--muted)" font-size="10">Social Media</text>
            <text x="80" y="140" text-anchor="middle" fill="var(--muted)" font-size="10">Alumni Pages</text>
          </g>
          
          <!-- Arrow 1 -->
          <path d="M 150 100 L 230 100" stroke="url(#flowGradient)" stroke-width="2" fill="none" marker-end="url(#arrowhead)"/>
          
          <!-- Zeal Engine -->
          <g class="diagram__section">
            <rect x="240" y="60" width="120" height="80" rx="8" fill="var(--accent)" fill-opacity="0.1" stroke="var(--accent)"/>
            <text x="300" y="85" text-anchor="middle" fill="var(--accent)" font-size="14" font-weight="600">Zeal Engine</text>
            <text x="300" y="105" text-anchor="middle" fill="var(--text)" font-size="10">AI Processing</text>
            <text x="300" y="120" text-anchor="middle" fill="var(--text)" font-size="10">Personality Analysis</text>
          </g>
          
          <!-- Arrow 2 -->
          <path d="M 370 100 L 450 100" stroke="url(#flowGradient)" stroke-width="2" fill="none" marker-end="url(#arrowhead)"/>
          
          <!-- CRM -->
          <g class="diagram__section">
            <rect x="460" y="40" width="120" height="120" rx="8" fill="var(--panel)" stroke="var(--muted)"/>
            <text x="520" y="60" text-anchor="middle" fill="var(--text)" font-size="14" font-weight="600">Your CRM</text>
            <text x="520" y="80" text-anchor="middle" fill="var(--muted)" font-size="10">Enriched Profiles</text>
            <text x="520" y="95" text-anchor="middle" fill="var(--muted)" font-size="10">Personality Palette</text>
            <text x="520" y="110" text-anchor="middle" fill="var(--muted)" font-size="10">Tailored Outreach</text>
            <text x="520" y="125" text-anchor="middle" fill="var(--muted)" font-size="10">Meeting Notes</text>
            <text x="520" y="140" text-anchor="middle" fill="var(--muted)" font-size="10">Source Links</text>
          </g>
          
          <!-- Arrow markers -->
          <defs>
            <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
              <polygon points="0 0, 10 3.5, 0 7" fill="var(--accent)" />
            </marker>
          </defs>
        </svg>
      </div>
    </div>
  </section>
</template>

<script setup>
// No script needed for this component
</script>

<style scoped>
.how-it-works {
  background: linear-gradient(135deg, rgba(91, 215, 194, 0.05), rgba(122, 162, 255, 0.05));
}

.how-it-works__title {
  margin-bottom: 1rem;
}

.how-it-works__subtitle {
  font-size: 1.125rem;
  margin-bottom: 4rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.how-it-works__steps {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 3rem;
  margin-bottom: 4rem;
}

.step {
  text-align: center;
  padding: 2rem;
  background: var(--panel);
  border-radius: var(--border-radius-lg);
  border: 1px solid rgba(166, 176, 187, 0.1);
  transition: var(--transition);
}

.step:hover {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: var(--shadow-lg);
}

.step__icon {
  width: 64px;
  height: 64px;
  background: linear-gradient(135deg, var(--accent), var(--accent-2));
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1.5rem;
  color: var(--bg);
}

.step__title {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: var(--text);
}

.step__description {
  color: var(--muted);
  line-height: 1.6;
}

.how-it-works__diagram {
  background: var(--panel);
  border-radius: var(--border-radius-lg);
  padding: 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
}

.diagram {
  width: 100%;
  height: auto;
  max-width: 800px;
  margin: 0 auto;
  display: block;
}

@media (max-width: 768px) {
  .how-it-works__steps {
    grid-template-columns: 1fr;
    gap: 2rem;
  }
  
  .step {
    padding: 1.5rem;
  }
  
  .diagram {
    font-size: 12px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .step:hover {
    transform: none;
  }
}
</style>

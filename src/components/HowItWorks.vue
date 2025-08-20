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
        <svg class="diagram" viewBox="0 0 1100 350" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <!-- Modern gradient flows -->
            <linearGradient id="flowGradient" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%" style="stop-color:#5BDBC2;stop-opacity:1" />
              <stop offset="50%" style="stop-color:#7AA2FF;stop-opacity:1" />
              <stop offset="100%" style="stop-color:#5BDBC2;stop-opacity:1" />
            </linearGradient>
            
            <!-- Urban glass card effect -->
            <linearGradient id="cardGradient" x1="0%" y1="0%" x2="100%" y2="100%">
              <stop offset="0%" style="stop-color:rgba(255,255,255,0.08);stop-opacity:1" />
              <stop offset="100%" style="stop-color:rgba(255,255,255,0.02);stop-opacity:1" />
            </linearGradient>
            
            <!-- Modern neon glow -->
            <filter id="neonGlow">
              <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
              <feMerge> 
                <feMergeNode in="coloredBlur"/>
                <feMergeNode in="SourceGraphic"/>
              </feMerge>
            </filter>
            
            <!-- Sharp modern arrow -->
            <marker id="arrowhead" markerWidth="14" markerHeight="10" 
                    refX="13" refY="5" orient="auto">
              <path d="M 0 0 L 14 5 L 0 10 L 3 5 Z" fill="url(#flowGradient)" />
            </marker>
            
            <!-- Urban grid pattern -->
            <pattern id="urbanGrid" width="20" height="20" patternUnits="userSpaceOnUse">
              <path d="M 20 0 L 0 0 0 20" fill="none" stroke="rgba(91, 219, 194, 0.03)" stroke-width="0.5"/>
              <circle cx="10" cy="10" r="0.5" fill="rgba(91, 219, 194, 0.05)"/>
            </pattern>
            
            <!-- Modern shadow -->
            <filter id="modernShadow">
              <feDropShadow dx="0" dy="8" stdDeviation="12" flood-color="rgba(0,0,0,0.15)"/>
            </filter>
          </defs>
          
          <!-- Urban background -->
          <rect width="100%" height="100%" fill="url(#urbanGrid)" />
          
          <!-- Data nexus lines -->
          <g opacity="0.1">
            <path d="M 120 60 Q 400 30 680 60" stroke="var(--accent-2)" stroke-width="1" fill="none"/>
            <path d="M 120 290 Q 400 320 680 290" stroke="var(--accent-2)" stroke-width="1" fill="none"/>
          </g>
          
          <!-- Step 1: Data Collection -->
          <g class="diagram__step" filter="url(#modernShadow)">
            <rect x="30" y="60" width="180" height="200" rx="16" fill="url(#cardGradient)" stroke="rgba(91, 219, 194, 0.4)" stroke-width="2"/>
            <!-- Modern header accent -->
            <rect x="30" y="60" width="180" height="4" rx="16" fill="var(--accent-2)" opacity="0.6"/>
            
            <text x="120" y="85" text-anchor="middle" fill="#5BDBC2" font-size="15" font-weight="800">01</text>
            <text x="120" y="105" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">DATA COLLECTION</text>
            <text x="120" y="120" text-anchor="middle" fill="var(--muted)" font-size="10" opacity="0.8">Multi-Source Intelligence</text>
            
            <!-- Modern data nodes -->
            <g class="data-sources">
              <!-- Hexagonal nodes for modern tech feel -->
              <g transform="translate(80, 135)">
                <polygon points="-12,0 -6,-10 6,-10 12,0 6,10 -6,10" fill="rgba(91, 219, 194, 0.15)" stroke="#5BDBC2" stroke-width="1"/>
                <circle cx="0" cy="0" r="4" fill="#5BDBC2"/>
                <text x="0" y="25" text-anchor="middle" fill="var(--muted)" font-size="8">WEB</text>
              </g>
              
              <g transform="translate(160, 135)">
                <polygon points="-12,0 -6,-10 6,-10 12,0 6,10 -6,10" fill="rgba(91, 219, 194, 0.15)" stroke="#5BDBC2" stroke-width="1"/>
                <circle cx="0" cy="0" r="4" fill="#5BDBC2"/>
                <text x="0" y="25" text-anchor="middle" fill="var(--muted)" font-size="8">AUDIO</text>
              </g>
              
              <g transform="translate(80, 190)">
                <polygon points="-12,0 -6,-10 6,-10 12,0 6,10 -6,10" fill="rgba(91, 219, 194, 0.15)" stroke="#5BDBC2" stroke-width="1"/>
                <circle cx="0" cy="0" r="4" fill="#5BDBC2"/>
                <text x="0" y="25" text-anchor="middle" fill="var(--muted)" font-size="8">NEWS</text>
              </g>
              
              <g transform="translate(160, 190)">
                <polygon points="-12,0 -6,-10 6,-10 12,0 6,10 -6,10" fill="rgba(91, 219, 194, 0.15)" stroke="#5BDBC2" stroke-width="1"/>
                <circle cx="0" cy="0" r="4" fill="#5BDBC2"/>
                <text x="0" y="25" text-anchor="middle" fill="var(--muted)" font-size="8">SOCIAL</text>
              </g>
              
              <g transform="translate(120, 240)">
                <polygon points="-12,0 -6,-10 6,-10 12,0 6,10 -6,10" fill="rgba(91, 219, 194, 0.15)" stroke="#5BDBC2" stroke-width="1"/>
                <circle cx="0" cy="0" r="4" fill="#5BDBC2"/>
                <text x="0" y="25" text-anchor="middle" fill="var(--muted)" font-size="8">ALUMNI</text>
              </g>
            </g>
          </g>
          
          <!-- Modern Data Flow Arrow -->
          <path d="M 225 160 Q 260 140 295 160" stroke="url(#flowGradient)" stroke-width="4" fill="none" marker-end="url(#arrowhead)" filter="url(#neonGlow)"/>
          <rect x="240" y="125" width="40" height="16" rx="8" fill="rgba(122, 162, 255, 0.2)" stroke="var(--accent)" stroke-width="1"/>
          <text x="260" y="135" text-anchor="middle" fill="var(--accent)" font-size="9" font-weight="700">DATA</text>
          
          <!-- Step 2: AI Processing Core -->
          <g class="diagram__step" filter="url(#modernShadow)">
            <rect x="310" y="40" width="180" height="240" rx="16" fill="url(#cardGradient)" stroke="rgba(122, 162, 255, 0.5)" stroke-width="2"/>
            <!-- Neural network accent -->
            <rect x="310" y="40" width="180" height="4" rx="16" fill="var(--accent)" opacity="0.8"/>
            
            <text x="400" y="65" text-anchor="middle" fill="#7AA2FF" font-size="15" font-weight="800">02</text>
            <text x="400" y="85" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">AI PROCESSING</text>
            <text x="400" y="100" text-anchor="middle" fill="var(--muted)" font-size="10" opacity="0.8">Zeal Intelligence Engine</text>
            
            <!-- Neural processing nodes -->
            <g class="processing-nodes">
              <!-- NLP Module -->
              <rect x="325" y="115" width="150" height="28" rx="14" fill="rgba(122, 162, 255, 0.15)" stroke="var(--accent)" stroke-width="1"/>
              <circle cx="340" cy="129" r="6" fill="var(--accent)" opacity="0.8"/>
              <text x="365" y="125" text-anchor="start" fill="var(--accent)" font-size="9" font-weight="700">NLP ANALYSIS</text>
              <text x="365" y="135" text-anchor="start" fill="var(--muted)" font-size="7">Extract insights</text>
              
              <!-- Pattern Recognition -->
              <rect x="325" y="150" width="150" height="28" rx="14" fill="rgba(122, 162, 255, 0.15)" stroke="var(--accent)" stroke-width="1"/>
              <circle cx="340" cy="164" r="6" fill="var(--accent)" opacity="0.8"/>
              <text x="365" y="160" text-anchor="start" fill="var(--accent)" font-size="9" font-weight="700">PATTERN RECOGNITION</text>
              <text x="365" y="170" text-anchor="start" fill="var(--muted)" font-size="7">Behavior analysis</text>
              
              <!-- Personality Mapping -->
              <rect x="325" y="185" width="150" height="28" rx="14" fill="rgba(122, 162, 255, 0.15)" stroke="var(--accent)" stroke-width="1"/>
              <circle cx="340" cy="199" r="6" fill="var(--accent)" opacity="0.8"/>
              <text x="365" y="195" text-anchor="start" fill="var(--accent)" font-size="9" font-weight="700">PERSONALITY MAP</text>
              <text x="365" y="205" text-anchor="start" fill="var(--muted)" font-size="7">Color palette</text>
              
              <!-- Quality Scoring -->
              <rect x="325" y="220" width="150" height="28" rx="14" fill="rgba(122, 162, 255, 0.15)" stroke="var(--accent)" stroke-width="1"/>
              <circle cx="340" cy="234" r="6" fill="var(--accent)" opacity="0.8"/>
              <text x="365" y="230" text-anchor="start" fill="var(--accent)" font-size="9" font-weight="700">QUALITY SCORE</text>
              <text x="365" y="240" text-anchor="start" fill="var(--muted)" font-size="7">Confidence metrics</text>
              
              <!-- Neural connections -->
              <g opacity="0.3">
                <path d="M 340 129 L 460 164" stroke="var(--accent)" stroke-width="1"/>
                <path d="M 340 164 L 460 199" stroke="var(--accent)" stroke-width="1"/>
                <path d="M 340 199 L 460 234" stroke="var(--accent)" stroke-width="1"/>
              </g>
            </g>
          </g>
          
          <!-- Enhanced Flow Arrow -->
          <path d="M 505 160 Q 540 140 575 160" stroke="url(#flowGradient)" stroke-width="4" fill="none" marker-end="url(#arrowhead)" filter="url(#neonGlow)"/>
          <rect x="520" y="125" width="60" height="16" rx="8" fill="rgba(91, 215, 194, 0.2)" stroke="var(--accent-2)" stroke-width="1"/>
          <text x="550" y="135" text-anchor="middle" fill="var(--accent-2)" font-size="9" font-weight="700">PROFILE</text>
          
          <!-- Step 3: CRM Integration Hub -->
          <g class="diagram__step" filter="url(#modernShadow)">
            <rect x="590" y="60" width="180" height="200" rx="16" fill="url(#cardGradient)" stroke="rgba(91, 215, 194, 0.4)" stroke-width="2"/>
            <!-- Integration accent -->
            <rect x="590" y="60" width="180" height="4" rx="16" fill="var(--accent-2)" opacity="0.6"/>
            
            <text x="680" y="85" text-anchor="middle" fill="#5BDBC2" font-size="15" font-weight="800">03</text>
            <text x="680" y="105" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">CRM INTEGRATION</text>
            <text x="680" y="120" text-anchor="middle" fill="var(--muted)" font-size="10" opacity="0.8">Actionable Intelligence</text>
            
            <!-- Modern CRM modules -->
            <g class="crm-modules">
              <rect x="605" y="135" width="150" height="24" rx="12" fill="rgba(91, 215, 194, 0.15)" stroke="var(--accent-2)" stroke-width="1"/>
              <rect x="615" y="139" width="4" height="16" rx="2" fill="var(--accent-2)"/>
              <text x="630" y="148" text-anchor="start" fill="var(--text)" font-size="9" font-weight="700">PERSONALITY PALETTE</text>
              
              <rect x="605" y="165" width="150" height="24" rx="12" fill="rgba(91, 215, 194, 0.15)" stroke="var(--accent-2)" stroke-width="1"/>
              <rect x="615" y="169" width="4" height="16" rx="2" fill="var(--accent-2)"/>
              <text x="630" y="178" text-anchor="start" fill="var(--text)" font-size="9" font-weight="700">TAILORED MESSAGING</text>
              
              <rect x="605" y="195" width="150" height="24" rx="12" fill="rgba(91, 215, 194, 0.15)" stroke="var(--accent-2)" stroke-width="1"/>
              <rect x="615" y="199" width="4" height="16" rx="2" fill="var(--accent-2)"/>
              <text x="630" y="208" text-anchor="start" fill="var(--text)" font-size="9" font-weight="700">MEETING INSIGHTS</text>
              
              <rect x="605" y="225" width="150" height="24" rx="12" fill="rgba(91, 215, 194, 0.15)" stroke="var(--accent-2)" stroke-width="1"/>
              <rect x="615" y="229" width="4" height="16" rx="2" fill="var(--accent-2)"/>
              <text x="630" y="238" text-anchor="start" fill="var(--text)" font-size="9" font-weight="700">SOURCE ATTRIBUTION</text>
            </g>
          </g>
          
          <!-- Success Flow Arrow -->
          <path d="M 785 160 Q 820 140 855 160" stroke="url(#flowGradient)" stroke-width="4" fill="none" marker-end="url(#arrowhead)" filter="url(#neonGlow)"/>
          <rect x="800" y="125" width="50" height="16" rx="8" fill="rgba(255, 193, 7, 0.2)" stroke="#FFD700" stroke-width="1"/>
          <text x="825" y="135" text-anchor="middle" fill="#FFD700" font-size="9" font-weight="700">SUCCESS</text>
          
          <!-- Step 4: Results Dashboard -->
          <g class="diagram__step" filter="url(#modernShadow)">
            <rect x="870" y="90" width="120" height="140" rx="16" fill="url(#cardGradient)" stroke="rgba(255, 193, 7, 0.5)" stroke-width="2"/>
            <!-- Success accent -->
            <rect x="870" y="90" width="120" height="4" rx="16" fill="#FFD700" opacity="0.8"/>
            
            <text x="930" y="115" text-anchor="middle" fill="#FFD700" font-size="15" font-weight="800">04</text>
            <text x="930" y="135" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">RESULTS</text>
            
            <!-- Success metrics -->
            <g class="success-metrics">
              <rect x="885" y="150" width="90" height="28" rx="14" fill="rgba(255, 193, 7, 0.15)" stroke="#FFD700" stroke-width="1"/>
              <circle cx="900" cy="164" r="6" fill="#FFD700"/>
              <text x="915" y="160" text-anchor="start" fill="#FFD700" font-size="8" font-weight="700">HIGHER</text>
              <text x="915" y="170" text-anchor="start" fill="#FFD700" font-size="8" font-weight="700">RESPONSE</text>
              
              <rect x="885" y="185" width="90" height="28" rx="14" fill="rgba(255, 193, 7, 0.15)" stroke="#FFD700" stroke-width="1"/>
              <circle cx="900" cy="199" r="6" fill="#FFD700"/>
              <text x="915" y="195" text-anchor="start" fill="#FFD700" font-size="8" font-weight="700">DEEPER</text>
              <text x="915" y="205" text-anchor="start" fill="#FFD700" font-size="8" font-weight="700">CONNECTIONS</text>
            </g>
          </g>
          
          <!-- Modern ambient elements -->
          <g class="ambient-tech" opacity="0.05">
            <circle cx="120" cy="30" r="20" fill="var(--accent-2)"/>
            <circle cx="400" cy="20" r="15" fill="var(--accent)"/>
            <circle cx="680" cy="30" r="22" fill="var(--accent-2)"/>
            <circle cx="930" cy="310" r="25" fill="#FFD700"/>
            <!-- Tech grid nodes -->
            <circle cx="200" cy="300" r="3" fill="var(--accent-2)"/>
            <circle cx="500" cy="320" r="3" fill="var(--accent)"/>
            <circle cx="800" cy="300" r="3" fill="var(--accent-2)"/>
          </g>
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
  background: var(--bg);
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
  background: transparent;
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
  background: transparent;
  border-radius: var(--border-radius-lg);
  padding: 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
  margin-top: var(--space-3xl);
  width: 100%;
  overflow-x: auto;
  overflow-y: hidden;
}

.diagram {
  width: 100%;
  height: auto;
  min-width: 1100px;
  margin: 0 auto;
  display: block;
}

@media (max-width: 1200px) {
  .how-it-works__diagram {
    margin-left: calc(-1 * var(--space-lg));
    margin-right: calc(-1 * var(--space-lg));
    border-left: none;
    border-right: none;
    border-radius: 0;
  }
}

@media (max-width: 768px) {
  .how-it-works__steps {
    grid-template-columns: 1fr;
    gap: 2rem;
  }
  
  .step {
    padding: 1.5rem;
  }
  
  .how-it-works__diagram {
    padding: 1rem;
  }
  
  .diagram {
    min-width: 800px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .step:hover {
    transform: none;
  }
}
</style>

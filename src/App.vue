<template>
  <div id="app">
    <Header />
    <Hero :persona="samplePersona" />
    <HowItWorks />
    <DemoComposer :persona="samplePersona" />
    <Integrations />
    <FeaturesGrid />
    <Pricing />
    <Testimonials />
    <FAQ />
    <Footer />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Header from './components/Header.vue'
import Hero from './components/Hero.vue'
import HowItWorks from './components/HowItWorks.vue'
import DemoComposer from './components/DemoComposer.vue'
import Integrations from './components/Integrations.vue'
import FeaturesGrid from './components/FeaturesGrid.vue'
import Pricing from './components/Pricing.vue'
import Testimonials from './components/Testimonials.vue'
import FAQ from './components/FAQ.vue'
import Footer from './components/Footer.vue'

const samplePersona = ref({
  "name": "Avery Chen",
  "role": "VP Sales, North America",
  "company": "Finch Robotics",
  "location": "San Francisco, CA",
  "email": "avery.chen@example.com",
  "recentSignals": [
    {"type": "podcast", "title": "RevOps Unpacked", "date": "2025-07-22", "note": "Emphasized data-driven experimentation"},
    {"type": "press", "title": "Finch Series B Extension", "date": "2025-06-30", "note": "Focus on enterprise expansion"},
    {"type": "event", "title": "SaaStr panel", "date": "2025-06-10", "note": "Prefers concise, metric-led pitches"}
  ],
  "traits": [
    {"name": "Analytical", "score": 0.84, "color": "#64a1ff"},
    {"name": "Direct", "score": 0.78, "color": "#a5ffda"},
    {"name": "Vision-led", "score": 0.66, "color": "#ffd166"},
    {"name": "Risk-averse", "score": 0.41, "color": "#ff9b9b"},
    {"name": "Detail-oriented", "score": 0.72, "color": "#b19cff"},
    {"name": "Empathic", "score": 0.58, "color": "#7bd8ff"}
  ],
  "tone": "concise, data-driven",
  "interests": ["RevOps", "AI enablement", "Enterprise playbooks"]
})
</script>

<style>
:root {
  --bg: #0b0d10;
  --panel: #111418;
  --text: #e8eef2;
  --muted: #a6b0bb;
  --accent: #5bd7c2;
  --accent-2: #7aa2ff;
  --warning: #f2b705;
  
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
  --font-mono: 'SF Mono', Monaco, 'Inconsolata', 'Roboto Mono', 'Source Code Pro', monospace;
  
  --border-radius: 8px;
  --border-radius-lg: 12px;
  --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  
  --container-max-width: 1200px;
  --container-padding: 1rem;
  
  --transition: all 0.2s ease;
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --transition: none;
  }
  
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  scroll-behavior: smooth;
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}

body {
  font-family: var(--font-family);
  background: var(--bg);
  color: var(--text);
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.container {
  max-width: var(--container-max-width);
  margin: 0 auto;
  padding: 0 var(--container-padding);
}

.grid {
  display: grid;
  gap: 2rem;
}

.grid-2 {
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}

.grid-3 {
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}

.grid-4 {
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}

@media (min-width: 768px) {
  .grid-2 {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .grid-3 {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .grid-4 {
    grid-template-columns: repeat(4, 1fr);
  }
}

.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: var(--border-radius);
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: var(--transition);
  white-space: nowrap;
}

.btn:focus {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.btn-primary {
  background: var(--accent);
  color: var(--bg);
}

.btn-primary:hover {
  background: #4cc4b2;
  transform: translateY(-1px);
}

.btn-secondary {
  background: transparent;
  color: var(--text);
  border: 1px solid var(--muted);
}

.btn-secondary:hover {
  border-color: var(--accent);
  background: rgba(91, 215, 194, 0.1);
}

.section {
  padding: 4rem 0;
}

.section-large {
  padding: 6rem 0;
}

h1, h2, h3, h4, h5, h6 {
  font-weight: 600;
  line-height: 1.2;
  margin-bottom: 1rem;
}

h1 {
  font-size: 3rem;
  font-weight: 700;
}

h2 {
  font-size: 2.25rem;
}

h3 {
  font-size: 1.875rem;
}

h4 {
  font-size: 1.5rem;
}

@media (max-width: 768px) {
  h1 {
    font-size: 2.25rem;
  }
  
  h2 {
    font-size: 1.875rem;
  }
  
  h3 {
    font-size: 1.5rem;
  }
}

p {
  margin-bottom: 1rem;
  color: var(--muted);
}

.text-center {
  text-align: center;
}

.text-large {
  font-size: 1.125rem;
}

.text-muted {
  color: var(--muted);
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

#app {
  min-height: 100vh;
}
</style>

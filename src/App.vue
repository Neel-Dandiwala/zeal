<template>
  <div id="app">
    <!-- Main Content -->
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
@import './styles/global.css';

#app {
  min-height: 100vh;
  position: relative;
  z-index: var(--z-content);
}

/* Ensure content is above the nebula background */
main {
  position: relative;
  z-index: var(--z-content);
}
</style>

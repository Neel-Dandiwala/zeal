<template>
  <header class="header" :class="{ 'header--scrolled': isScrolled }">
    <div class="container">
      <nav class="nav" role="navigation" aria-label="Main navigation">
        <div class="nav__brand">
          <a href="#" class="nav__logo" aria-label="Zeal home">
            <span class="nav__logo-text">Zeal</span>
          </a>
        </div>
        
        <div class="nav__menu" :class="{ 'nav__menu--open': isMenuOpen }">
          <ul class="nav__links">
            <li><a href="#product" class="nav__link">Product</a></li>
            <li><a href="#integrations" class="nav__link">Integrations</a></li>
            <li><a href="#how-it-works" class="nav__link">How it Works</a></li>
            <li><a href="#pricing" class="nav__link">Pricing</a></li>
            <li><a href="#faq" class="nav__link">FAQ</a></li>
            <li><a href="#contact" class="nav__link">Contact</a></li>
          </ul>
        </div>
        
        <div class="nav__actions">
          <a href="#trial" class="btn btn-ghost">Start Free Trial</a>
          <a href="#demo" class="btn btn-primary">Book Demo</a>
        </div>
        
        <button 
          class="nav__toggle"
          :aria-expanded="isMenuOpen"
          aria-controls="nav-menu"
          aria-label="Toggle navigation menu"
          @click="toggleMenu"
        >
          <span class="nav__toggle-line"></span>
          <span class="nav__toggle-line"></span>
          <span class="nav__toggle-line"></span>
        </button>
      </nav>
    </div>
  </header>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const isScrolled = ref(false)
const isMenuOpen = ref(false)

const handleScroll = () => {
  isScrolled.value = window.scrollY > 20
}

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})
</script>

<style scoped>
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: var(--z-header);
  background: rgba(0, 0, 0, 0.6);
  border-bottom: 1px solid var(--line);
  transition: all var(--dur-1) var(--e-out);
}

.header--scrolled {
  background: rgba(0, 0, 0, 0.9);
  border-bottom-color: rgba(255, 255, 255, 0.1);
}

.nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-md) 0;
  position: relative;
}

.nav__logo {
  text-decoration: none;
  font-size: var(--text-lg);
  font-weight: 700;
  color: var(--text);
  transition: color var(--dur-1) var(--e-out);
}

.nav__logo:hover {
  color: var(--accent);
}

.nav__logo-text {
  font-size: var(--text-xl);
}

.nav__menu {
  display: flex;
  align-items: center;
}

.nav__links {
  display: flex;
  list-style: none;
  gap: var(--space-xl);
  margin: 0;
  padding: 0;
}

.nav__link {
  color: var(--muted);
  text-decoration: none;
  font-weight: 500;
  font-size: var(--text-sm);
  position: relative;
  transition: color var(--dur-1) var(--e-out);
}

.nav__link:hover,
.nav__link:focus {
  color: var(--text);
}

/* Navigation underline using global nav system */
.nav__link::after {
  content: "";
  position: absolute;
  left: 0;
  right: 100%;
  bottom: -6px;
  height: 2px;
  background: var(--accent);
  transition: right var(--dur-1) var(--e-out);
}

.nav__link:hover::after,
.nav__link:focus-visible::after {
  right: 0;
}

.nav__actions {
  display: flex;
  align-items: center;
  gap: var(--space-md);
}

.nav__toggle {
  display: none;
  flex-direction: column;
  justify-content: space-around;
  width: 24px;
  height: 24px;
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
}

.nav__toggle:focus-visible {
  outline: none;
  box-shadow: 0 0 0 4px color-mix(in srgb, var(--accent) 35%, transparent);
  border-radius: var(--r-xs);
}

.nav__toggle-line {
  width: 100%;
  height: 2px;
  background: var(--text);
  transition: all var(--dur-1) var(--e-out);
  border-radius: 1px;
}

@media (max-width: 768px) {
  .nav__menu {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: rgba(0, 0, 0, 0.95);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: var(--r-md);
    margin-top: var(--space-md);
    padding: var(--space-lg);
    opacity: 0;
    visibility: hidden;
    transform: translateY(-10px);
    transition: all var(--dur-2) var(--e-out);
  }
  
  .nav__menu--open {
    opacity: 1;
    visibility: visible;
    transform: translateY(0);
  }
  
  .nav__links {
    flex-direction: column;
    gap: var(--space-md);
    width: 100%;
  }
  
  .nav__link {
    width: 100%;
    text-align: center;
    padding: var(--space-md);
  }
  
  .nav__actions {
    margin-top: var(--space-lg);
    justify-content: center;
    gap: var(--space-sm);
  }
  
  .nav__toggle {
    display: flex;
  }
  
  .nav__toggle[aria-expanded="true"] .nav__toggle-line:nth-child(1) {
    transform: rotate(45deg) translate(5px, 5px);
  }
  
  .nav__toggle[aria-expanded="true"] .nav__toggle-line:nth-child(2) {
    opacity: 0;
  }
  
  .nav__toggle[aria-expanded="true"] .nav__toggle-line:nth-child(3) {
    transform: rotate(-45deg) translate(7px, -6px);
  }
}

@media (max-width: 480px) {
  .nav__actions {
    flex-direction: column;
    gap: var(--space-sm);
  }
  
  .btn {
    width: 100%;
    justify-content: center;
  }
}
</style>

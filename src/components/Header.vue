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
          <a href="#trial" class="btn btn-secondary">Start Free Trial</a>
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
  z-index: 1000;
  background: rgba(11, 13, 16, 0.8);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid transparent;
  transition: var(--transition);
}

.header--scrolled {
  background: rgba(11, 13, 16, 0.95);
  border-bottom-color: rgba(166, 176, 187, 0.1);
  box-shadow: var(--shadow);
}

.nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 0;
  position: relative;
}

.nav__logo {
  text-decoration: none;
  color: var(--text);
  font-size: 1.5rem;
  font-weight: 700;
  transition: var(--transition);
}

.nav__logo:hover {
  color: var(--accent);
}

.nav__logo-text {
  background: linear-gradient(135deg, var(--accent), var(--accent-2));
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.nav__menu {
  display: flex;
  align-items: center;
}

.nav__links {
  display: flex;
  list-style: none;
  gap: 2rem;
  margin: 0;
  padding: 0;
}

.nav__link {
  color: var(--muted);
  text-decoration: none;
  font-weight: 500;
  transition: var(--transition);
  position: relative;
}

.nav__link:hover,
.nav__link:focus {
  color: var(--text);
}

.nav__link::after {
  content: '';
  position: absolute;
  bottom: -4px;
  left: 0;
  width: 0;
  height: 2px;
  background: var(--accent);
  transition: var(--transition);
}

.nav__link:hover::after,
.nav__link:focus::after {
  width: 100%;
}

.nav__actions {
  display: flex;
  align-items: center;
  gap: 1rem;
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

.nav__toggle-line {
  width: 100%;
  height: 2px;
  background: var(--text);
  transition: var(--transition);
}

@media (max-width: 768px) {
  .nav__menu {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: var(--panel);
    border: 1px solid rgba(166, 176, 187, 0.1);
    border-radius: var(--border-radius);
    margin-top: 1rem;
    padding: 1rem;
    opacity: 0;
    visibility: hidden;
    transform: translateY(-10px);
    transition: var(--transition);
  }
  
  .nav__menu--open {
    opacity: 1;
    visibility: visible;
    transform: translateY(0);
  }
  
  .nav__links {
    flex-direction: column;
    gap: 1rem;
  }
  
  .nav__actions {
    display: none;
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
</style>

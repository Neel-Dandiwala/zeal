# Zeal - B2B Landing Page

A high-conversion, dark-theme landing page for Zeal CRM built with Vue 3 and plain JavaScript.

## ✨ Features

- **Dark Theme**: WCAG-compliant dark design with CSS variables
- **Interactive Personality Palette**: Keyboard-accessible trait visualization with hover tooltips
- **Demo Composer**: Live text generation with tone switching (Concise/Analytical/Friendly)
- **Responsive Design**: Mobile-first 12-column grid system
- **Accessibility**: Full keyboard navigation, ARIA labels, reduced motion support
- **Performance**: Under 100KB JS bundle, lazy loading, optimized images
- **SEO Ready**: Meta tags, Open Graph, JSON-LD structured data

## 🚀 Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## 🎨 Design System

The landing page uses a consistent design system with CSS variables:

```css
--bg: #0b0d10;        /* Background */
--panel: #111418;     /* Panel background */
--text: #e8eef2;      /* Primary text */
--muted: #a6b0bb;     /* Secondary text */
--accent: #5bd7c2;    /* Primary accent */
--accent-2: #7aa2ff;  /* Secondary accent */
--warning: #f2b705;   /* Warning/highlight */
```

## 🧩 Components

- **Header.vue**: Sticky navigation with mobile menu
- **Hero.vue**: Main hero section with personality palette
- **PersonalityPalette.vue**: Interactive trait visualization
- **HowItWorks.vue**: 3-step process explanation
- **DemoComposer.vue**: Live outreach generation demo
- **Integrations.vue**: Partner logos and features
- **FeaturesGrid.vue**: 6-card feature showcase
- **Pricing.vue**: 3-tier pricing with FAQ
- **Testimonials.vue**: Customer quotes and metrics
- **FAQ.vue**: Collapsible questions and answers
- **Footer.vue**: Links, contact, and social media

## 🎯 Key Interactions

### Personality Palette
- **Keyboard Navigation**: Arrow keys to navigate tiles
- **Tooltips**: Hover/focus shows trait details and sources
- **Accessibility**: Full ARIA support and screen reader compatibility

### Demo Composer
- **Live Generation**: Realistic sample text generation
- **Tone Switching**: Adjust output style (Concise/Analytical/Friendly)
- **Channel Support**: Email, LinkedIn DM, Meeting Notes
- **Copy to Clipboard**: One-click text copying

## ♿ Accessibility

- **WCAG 2.1 AA**: Compliant contrast ratios and interaction patterns
- **Keyboard Navigation**: All interactive elements accessible via keyboard
- **Screen Readers**: Proper ARIA labels and semantic markup
- **Reduced Motion**: Respects `prefers-reduced-motion` setting
- **Focus Management**: Visible focus indicators throughout

## 🔧 Performance Optimizations

- **Code Splitting**: Automatic chunking via Vite
- **Lazy Loading**: Below-the-fold images and components
- **CSS Variables**: Efficient theming and design tokens
- **Minimal Dependencies**: Vue 3 only, no external UI libraries
- **Bundle Size**: Target <100KB JavaScript (excluding Vue runtime)

## 📱 Browser Support

- Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Mobile Safari and Chrome
- Progressive enhancement for older browsers

## 🔒 Security & Privacy

- **No External APIs**: All demo functionality is client-side only
- **CSP Ready**: Compatible with Content Security Policy
- **Privacy First**: No tracking or analytics by default

## 📊 Lighthouse Scores

Target scores for production build:
- **Performance**: ≥90
- **Accessibility**: ≥90
- **Best Practices**: ≥90
- **SEO**: ≥90

## 🛠️ Development Notes

### Component Architecture
- Uses Vue 3 Composition API with `<script setup>`
- Props-down, events-up data flow
- Minimal state management (no Vuex/Pinia needed)

### Styling Approach
- CSS Variables for theming
- BEM-inspired class naming
- Mobile-first responsive design
- No CSS frameworks (Tailwind, Bootstrap, etc.)

### Code Quality
- ESLint + Prettier configuration
- Semantic HTML structure
- Accessible component patterns

## 📝 Content Strategy

The landing page copy emphasizes:
- **Data-driven benefits** with specific metrics
- **Personality-focused messaging** aligned with the product
- **Technical credibility** through source transparency
- **Enterprise readiness** with compliance and security mentions

## 🚢 Deployment

The production build generates static files suitable for any hosting:

```bash
npm run build
# Files output to dist/ directory
```

Deploy to:
- **Vercel**: Zero-config deployment
- **Netlify**: Drag-and-drop or Git integration
- **AWS S3 + CloudFront**: Static hosting
- **GitHub Pages**: Free hosting option

## 📄 License

MIT License - see LICENSE file for details.

---

Built with ❤️ using Vue 3, designed for conversion and accessibility.

<template>
  <div class="integration-diagram">
    <svg
      ref="svgRef"
      class="diagram-svg"
      viewBox="0 0 1400 300"
      role="img"
      :aria-label="ariaLabel"
      @mouseleave="handleMouseLeave"
    >
      <!-- Definitions for gradients, glows, and markers -->
      <defs>
        <!-- Zeal glow gradient -->
        <radialGradient id="zealGlow" cx="50%" cy="50%" r="50%">
          <stop offset="0%" style="stop-color:#2EF2E8;stop-opacity:1" />
          <stop offset="70%" style="stop-color:#6DE1F5;stop-opacity:0.8" />
          <stop offset="100%" style="stop-color:#2EF2E8;stop-opacity:0" />
        </radialGradient>
        
        <!-- Path flow gradient -->
        <linearGradient id="flowGradient" x1="0%" y1="0%" x2="100%" y2="0%">
          <stop offset="0%" style="stop-color:transparent" />
          <stop offset="40%" style="stop-color:#2EF2E8;stop-opacity:0.6" />
          <stop offset="60%" style="stop-color:#6DE1F5;stop-opacity:0.8" />
          <stop offset="100%" style="stop-color:transparent" />
        </linearGradient>
        
        <!-- Node glow filter -->
        <filter id="nodeGlow" x="-50%" y="-50%" width="200%" height="200%">
          <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
          <feMerge> 
            <feMergeNode in="coloredBlur"/>
            <feMergeNode in="SourceGraphic"/>
          </feMerge>
        </filter>
        
        <!-- Packet glow filter -->
        <filter id="packetGlow" x="-100%" y="-100%" width="300%" height="300%">
          <feGaussianBlur stdDeviation="4" result="coloredBlur"/>
          <feMerge> 
            <feMergeNode in="coloredBlur"/>
            <feMergeNode in="SourceGraphic"/>
          </feMerge>
        </filter>
        
        <!-- Pulse ring animation -->
        <filter id="pulseGlow" x="-100%" y="-100%" width="300%" height="300%">
          <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
          <feMerge> 
            <feMergeNode in="coloredBlur"/>
            <feMergeNode in="SourceGraphic"/>
          </feMerge>
        </filter>
      </defs>
      
      <!-- Background paths (static) -->
      <g class="path-group background-paths">
        <path
          v-for="path in allPaths"
          :key="`bg-${path.id}`"
          :d="path.d"
          class="path-background"
          :class="{ 'path-highlighted': highlightedPaths.includes(path.id) }"
        />
      </g>
      
      <!-- Flow paths (animated) -->
      <g class="path-group flow-paths">
        <path
          v-for="path in allPaths"
          :key="`flow-${path.id}`"
          :d="path.d"
          class="path-flow"
          :class="{ 
            'path-highlighted': highlightedPaths.includes(path.id),
            'path-reduced-motion': effectiveReducedMotion
          }"
        />
      </g>
      
      <!-- Animated packet -->
      <circle
        v-if="packetPosition && !effectiveReducedMotion"
        :cx="packetPosition.x"
        :cy="packetPosition.y"
        r="4"
        class="packet"
        fill="url(#zealGlow)"
        filter="url(#packetGlow)"
      />
      
      <!-- Nodes -->
      <g class="nodes-group">
        <!-- Inbound nodes -->
        <g
          v-for="(node, index) in inbound"
          :key="`inbound-${node.id}`"
          class="node-group inbound-node"
          :class="{ 'node-highlighted': highlightedNodes.includes(node.id) }"
          :transform="`translate(${inboundPositions[index].x}, ${inboundPositions[index].y})`"
          tabindex="0"
          role="button"
          :aria-label="`${node.label} source integration`"
          @mouseenter="handleNodeHover(node, 'inbound')"
          @click="handleNodeClick(node, 'inbound')"
          @keydown="handleNodeKeydown($event, node, 'inbound')"
          @focus="handleNodeFocus(node, 'inbound')"
          @blur="handleNodeBlur"
        >
          <circle
            :r="nodeRadius"
            :fill="node.brandColor || '#4A5568'"
            class="node-bg"
            filter="url(#nodeGlow)"
          />
          <g class="node-content">
            <component
              :is="getLogoComponent(node)"
              v-bind="getLogoProps(node)"
            />
          </g>
        </g>
        
        <!-- Center Zeal node -->
        <g
          class="node-group center-node"
          :class="{ 'node-highlighted': highlightedNodes.includes(center.id) }"
          :transform="`translate(${centerPosition.x}, ${centerPosition.y})`"
          tabindex="0"
          role="button"
          :aria-label="center.label"
          @mouseenter="handleNodeHover(center, 'center')"
          @click="handleNodeClick(center, 'center')"
          @keydown="handleNodeKeydown($event, center, 'center')"
          @focus="handleNodeFocus(center, 'center')"
          @blur="handleNodeBlur"
        >
          <!-- Pulse ring -->
          <circle
            v-if="showPulse && !effectiveReducedMotion"
            :r="centerRadius + 10"
            fill="none"
            stroke="url(#zealGlow)"
            stroke-width="2"
            class="pulse-ring"
            filter="url(#pulseGlow)"
          />
          <circle
            :r="centerRadius"
            fill="url(#zealGlow)"
            class="center-bg"
            filter="url(#nodeGlow)"
          />
          <g class="node-content">
            <component
              :is="getLogoComponent(center)"
              v-bind="getLogoProps(center)"
            />
          </g>
        </g>
        
        <!-- Outbound nodes -->
        <g
          v-for="(node, index) in outbound"
          :key="`outbound-${node.id}`"
          class="node-group outbound-node"
          :class="{ 'node-highlighted': highlightedNodes.includes(node.id) }"
          :transform="`translate(${outboundPositions[index].x}, ${outboundPositions[index].y})`"
          tabindex="0"
          role="button"
          :aria-label="`${node.label} destination integration`"
          @mouseenter="handleNodeHover(node, 'outbound')"
          @click="handleNodeClick(node, 'outbound')"
          @keydown="handleNodeKeydown($event, node, 'outbound')"
          @focus="handleNodeFocus(node, 'outbound')"
          @blur="handleNodeBlur"
        >
          <circle
            :r="nodeRadius"
            :fill="node.brandColor || '#4A5568'"
            class="node-bg"
            filter="url(#nodeGlow)"
          />
          <g class="node-content">
            <component
              :is="getLogoComponent(node)"
              v-bind="getLogoProps(node)"
            />
          </g>
        </g>
      </g>
    </svg>
    
    <!-- Tooltip -->
    <div
      v-if="tooltip.visible"
      ref="tooltipRef"
      class="tooltip"
      :style="tooltipStyle"
      role="tooltip"
      :aria-live="tooltip.locked ? 'off' : 'polite'"
    >
      {{ tooltip.text }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch, nextTick } from 'vue'

// Type definitions
const props = defineProps({
  center: {
    type: Object,
    required: true
  },
  inbound: {
    type: Array,
    required: true
  },
  outbound: {
    type: Array,
    required: true
  },
  packetSpeed: {
    type: Number,
    default: 240
  },
  loopDelayMs: {
    type: Number,
    default: 400
  },
  reducedMotion: {
    type: Boolean,
    default: null
  }
})

// Reactive refs
const svgRef = ref(null)
const tooltipRef = ref(null)
const highlightedNodes = ref([])
const highlightedPaths = ref([])
const tooltip = ref({
  visible: false,
  text: '',
  x: 0,
  y: 0,
  locked: false
})
const packetPosition = ref(null)
const showPulse = ref(false)

// Animation state
let animationId = null
let packetAnimation = {
  currentPath: null,
  currentPathIndex: 0,
  startTime: 0,
  phase: 'inbound', // 'inbound', 'center', 'outbound'
  isAtCenter: false
}

// Computed properties
const centerPosition = computed(() => ({ x: 700, y: 150 }))
const nodeRadius = computed(() => 28)
const centerRadius = computed(() => 40)

const effectiveReducedMotion = computed(() => {
  if (props.reducedMotion !== null) return props.reducedMotion
  return window.matchMedia('(prefers-reduced-motion: reduce)').matches
})

const ariaLabel = computed(() => {
  const inboundNames = props.inbound.map(n => n.label).join(', ')
  const outboundNames = props.outbound.map(n => n.label).join(', ')
  return `Integration diagram showing data flow from ${inboundNames} through Zeal to ${outboundNames}`
})

// Calculate positions - horizontal layout spanning full width
const inboundPositions = computed(() => {
  const positions = []
  const baseY = 150
  const leftSideX = 120
  
  // Arrange inbound sources vertically on the left with proper spacing to avoid overlaps
  props.inbound.forEach((_, index) => {
    const yOffset = (index - (props.inbound.length - 1) / 2) * 60
    
    positions.push({
      x: leftSideX,
      y: baseY + yOffset
    })
  })
  
  return positions
})

const outboundPositions = computed(() => {
  const positions = []
  const baseY = 150
  const rightSideX = 1280
  
  // Arrange outbound destinations vertically on the right with proper spacing to avoid overlaps
  props.outbound.forEach((_, index) => {
    const yOffset = (index - (props.outbound.length - 1) / 2) * 60
    
    positions.push({
      x: rightSideX,
      y: baseY + yOffset
    })
  })
  
  return positions
})

// Generate paths
const allPaths = computed(() => {
  const paths = []
  const center = centerPosition.value
  
  // Inbound paths
  inboundPositions.value.forEach((pos, index) => {
    const node = props.inbound[index]
    const path = createCurvedPath(pos, center, 'inward')
    paths.push({
      id: `inbound-${node.id}`,
      d: path.d,
      element: path.element,
      length: path.length,
      type: 'inbound',
      nodeId: node.id
    })
  })
  
  // Outbound paths
  outboundPositions.value.forEach((pos, index) => {
    const node = props.outbound[index]
    const path = createCurvedPath(center, pos, 'outward')
    paths.push({
      id: `outbound-${node.id}`,
      d: path.d,
      element: path.element,
      length: path.length,
      type: 'outbound',
      nodeId: node.id
    })
  })
  
  return paths
})

// Create curved path between two points - horizontal flow
const createCurvedPath = (start, end, direction) => {
  const midX = (start.x + end.x) / 2
  const midY = (start.y + end.y) / 2
  
  // For horizontal flow, create gentle curves
  const curveFactor = 0.2
  const dx = end.x - start.x
  const dy = end.y - start.y
  
  // Control points for smooth horizontal flow
  const controlX1 = start.x + dx * curveFactor
  const controlY1 = start.y
  const controlX2 = end.x - dx * curveFactor  
  const controlY2 = end.y
  
  // Use cubic Bezier for smoother curves
  const d = `M ${start.x} ${start.y} C ${controlX1} ${controlY1}, ${controlX2} ${controlY2}, ${end.x} ${end.y}`
  
  // Create temporary path element to get length
  const tempPath = document.createElementNS('http://www.w3.org/2000/svg', 'path')
  tempPath.setAttribute('d', d)
  
  return {
    d,
    element: tempPath,
    length: tempPath.getTotalLength()
  }
}

// Logo components with official company SVG logos
const SimpleIconLogo = {
  props: ['slug', 'size'],
  template: `
    <g class="company-logo">
      <circle r="22" fill="white" opacity="0.08"/>
      <g transform="scale(0.6)">
        <!-- HubSpot Official Logo -->
        <g v-if="slug === 'hubspot'" fill="#FF7A59" transform="translate(-12, -12)">
          <path d="M12 0c6.627 0 12 5.373 12 12s-5.373 12-12 12S0 18.627 0 12 5.373 0 12 0zm0 2.4c-5.302 0-9.6 4.298-9.6 9.6s4.298 9.6 9.6 9.6 9.6-4.298 9.6-9.6S17.302 2.4 12 2.4zm0 3.6c3.314 0 6 2.686 6 6s-2.686 6-6 6-6-2.686-6-6 2.686-6 6-6z"/>
        </g>
        
        <!-- Salesforce Official Logo -->
        <g v-else-if="slug === 'salesforce'" fill="#00A1E0" transform="translate(-12, -12)">
          <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
          <circle cx="12" cy="12" r="8" fill="none" stroke="#00A1E0" stroke-width="1.5"/>
        </g>
        
        <!-- Apollo Logo -->
        <g v-else-if="slug === 'apollo'" fill="#6C5CE7" transform="translate(-12, -12)">
          <circle cx="12" cy="8" r="3"/>
          <path d="M6 16l6-8 6 8H6z"/>
        </g>
        
        <!-- Clay Logo -->
        <g v-else-if="slug === 'clay'" fill="#00B894" transform="translate(-12, -12)">
          <rect x="4" y="4" width="16" height="16" rx="4"/>
          <text x="12" y="15" text-anchor="middle" fill="white" font-size="10" font-weight="bold">C</text>
        </g>
        
        <!-- Google Official Logo -->
        <g v-else-if="slug === 'google'" transform="translate(-12, -12)">
          <path fill="#4285F4" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
          <path fill="#34A853" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
          <path fill="#FBBC05" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l3.66-2.84z"/>
          <path fill="#EA4335" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
        </g>
        
        <!-- Bing Official Logo -->
        <g v-else-if="slug === 'bing'" fill="#0078D4" transform="translate(-12, -12)">
          <path d="M5.5 3L7 4.5V12l5.5 3.5L18 13V9.5L12.5 7L7 9.5V7L5.5 3z"/>
        </g>
        
        <!-- Gmail Official Logo -->
        <g v-else-if="slug === 'gmail'" transform="translate(-12, -12)">
          <path fill="#EA4335" d="M24 5.25v13.5c0 .69-.56 1.25-1.25 1.25H1.25C.56 20 0 19.44 0 18.75V5.25C0 4.56.56 4 1.25 4h21.5C23.44 4 24 4.56 24 5.25z"/>
          <path fill="#FBBC05" d="M0 7.5L12 15l12-7.5"/>
          <path fill="#34A853" d="M0 5.25L12 12.75L24 5.25"/>
        </g>
        
        <!-- Slack Official Logo -->
        <g v-else-if="slug === 'slack'" transform="translate(-12, -12)">
          <path fill="#E01E5A" d="M8.5 12.5c0 .8-.7 1.5-1.5 1.5s-1.5-.7-1.5-1.5.7-1.5 1.5-1.5h1.5v1.5z"/>
          <path fill="#36C5F0" d="M10.75 8.5c-.8 0-1.5-.7-1.5-1.5s.7-1.5 1.5-1.5 1.5.7 1.5 1.5v1.5h-1.5z"/>
          <path fill="#2EB67D" d="M15.25 10.75c0-.8.7-1.5 1.5-1.5s1.5.7 1.5 1.5-.7 1.5-1.5 1.5h-1.5v-1.5z"/>
          <path fill="#ECB22E" d="M13 15.25c.8 0 1.5.7 1.5 1.5s-.7 1.5-1.5 1.5-1.5-.7-1.5-1.5v-1.5H13z"/>
        </g>
        
        <!-- Discord Official Logo -->
        <g v-else-if="slug === 'discord'" fill="#5865F2" transform="translate(-12, -12)">
          <path d="M19.27 5.33C17.94 4.71 16.5 4.26 15 4a.09.09 0 0 0-.07.03c-.18.33-.39.76-.53 1.09a16.09 16.09 0 0 0-4.8 0c-.14-.34-.35-.76-.54-1.09-.01-.02-.04-.03-.07-.03-1.5.26-2.93.71-4.27 1.33-.01 0-.02.01-.03.02-2.72 4.07-3.47 8.03-3.1 11.95 0 .02.01.04.03.05 1.8 1.32 3.53 2.12 5.24 2.65.03.01.06 0 .07-.02.4-.55.76-1.13 1.07-1.74.02-.04 0-.08-.04-.09-.57-.22-1.11-.48-1.64-.78-.04-.02-.04-.08-.01-.11.11-.08.22-.17.33-.25.02-.02.05-.02.07-.01 3.44 1.57 7.15 1.57 10.55 0 .02-.01.05-.01.07.01.11.09.22.17.33.26.04.03.04.09-.01.11-.52.31-1.07.56-1.64.78-.04.01-.05.06-.04.09.32.61.68 1.19 1.07 1.74.03.01.06.02.09.01 1.72-.53 3.45-1.33 5.25-2.65.02-.01.03-.03.03-.05.44-4.53-.73-8.46-3.1-11.95-.01-.01-.02-.02-.04-.02z"/>
        </g>
        
        <!-- WhatsApp Official Logo -->
        <g v-else-if="slug === 'whatsapp'" fill="#25D366" transform="translate(-12, -12)">
          <path d="M12.04 2c-5.46 0-9.91 4.45-9.91 9.91 0 1.75.46 3.45 1.32 4.95L2.05 22l5.25-1.38c1.45.79 3.08 1.21 4.74 1.21 5.46 0 9.91-4.45 9.91-9.91S17.5 2 12.04 2m.01 1.67c4.62 0 8.24 3.62 8.24 8.24 0 4.62-3.62 8.24-8.24 8.24-1.37 0-2.74-.35-3.96-1.02l-.54-.32-1.54.4.42-1.52-.36-.58c-.66-1.07-1.01-2.3-1.01-3.58 0-4.62 3.62-8.24 8.24-8.24"/>
        </g>
        
        <!-- LinkedIn Official Logo -->
        <g v-else-if="slug === 'linkedin'" fill="#0A66C2" transform="translate(-12, -12)">
          <path d="M20.5 2h-17A1.5 1.5 0 002 3.5v17A1.5 1.5 0 003.5 22h17a1.5 1.5 0 001.5-1.5v-17A1.5 1.5 0 0020.5 2zM8 19H5v-9h3zM6.5 8.25A1.75 1.75 0 118.3 6.5a1.78 1.78 0 01-1.8 1.75zM19 19h-3v-4.74c0-1.42-.6-1.93-1.38-1.93A1.74 1.74 0 0013 14.19a.66.66 0 000 .14V19h-3v-9h2.9v1.3a3.11 3.11 0 012.7-1.4c1.55 0 3.36.86 3.36 3.66z"/>
        </g>
        
        <!-- Globe/Web Icon -->
        <g v-else-if="slug === 'globe'" fill="#6B7280" transform="translate(-12, -12)">
          <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 17.93c-3.94-.49-7-3.85-7-7.93 0-.62.08-1.21.21-1.79L9 15v1c0 1.1.9 2 2 2v1.93zm6.9-2.54c-.26-.81-1-1.39-1.9-1.39h-1v-3c0-.55-.45-1-1-1H8v-2h2c.55 0 1-.45 1-1V7h2c1.1 0 2-.9 2-2v-.41c2.93 1.19 5 4.06 5 7.41 0 2.08-.8 3.97-2.1 5.39z"/>
        </g>
        
        <!-- Fallback Initial Letter -->
        <text v-else x="0" y="6" text-anchor="middle" dominant-baseline="central" fill="white" font-size="16" font-weight="700" font-family="system-ui">
          {{ slug.charAt(0).toUpperCase() }}
        </text>
      </g>
    </g>
  `
}

const UrlLogo = {
  props: ['src', 'size'],
  template: `
    <image 
      :href="src" 
      x="-12" 
      y="-12" 
      width="24" 
      height="24"
      preserveAspectRatio="xMidYMid meet"
    />
  `
}

const MonogramLogo = {
  props: ['text', 'size'],
  template: `
    <text
      x="0"
      y="0"
      text-anchor="middle"
      dominant-baseline="central"
      class="monogram-text"
      :style="{ fontSize: (size || 14) + 'px' }"
    >
      {{ text }}
    </text>
  `
}

const getLogoComponent = (node) => {
  switch (node.logo.kind) {
    case 'simpleIcon': return SimpleIconLogo
    case 'url': return UrlLogo
    case 'mono': return MonogramLogo
    default: return MonogramLogo
  }
}

const getLogoProps = (node) => {
  switch (node.logo.kind) {
    case 'simpleIcon': 
      return { slug: node.logo.slug, size: 14 }
    case 'url': 
      return { src: node.logo.src, size: 24 }
    case 'mono': 
      return { text: node.logo.text, size: 14 }
    default: 
      return { text: node.label.charAt(0), size: 14 }
  }
}

// Tooltip positioning
const tooltipStyle = computed(() => ({
  transform: `translate(${tooltip.value.x}px, ${tooltip.value.y}px)`,
  pointerEvents: tooltip.value.locked ? 'auto' : 'none'
}))

// Event handlers
const handleNodeHover = (node, type) => {
  if (tooltip.value.locked) return
  
  updateTooltip(node, type)
  highlightNode(node, type)
}

const handleNodeClick = (node, type) => {
  tooltip.value.locked = !tooltip.value.locked
  
  if (tooltip.value.locked) {
    updateTooltip(node, type)
    highlightNode(node, type)
    
    // Auto-unlock after 5 seconds
    setTimeout(() => {
      if (tooltip.value.locked) {
        tooltip.value.locked = false
        clearHighlights()
        hideTooltip()
      }
    }, 5000)
  } else {
    clearHighlights()
    hideTooltip()
  }
  
  if (node.href) {
    window.open(node.href, '_blank')
  }
}

const handleNodeKeydown = (event, node, type) => {
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault()
    handleNodeClick(node, type)
  } else if (event.key === 'Escape') {
    tooltip.value.locked = false
    clearHighlights()
    hideTooltip()
  }
}

const handleNodeFocus = (node, type) => {
  if (!tooltip.value.locked) {
    updateTooltip(node, type)
    highlightNode(node, type)
  }
}

const handleNodeBlur = () => {
  if (!tooltip.value.locked) {
    clearHighlights()
    hideTooltip()
  }
}

const handleMouseLeave = () => {
  if (!tooltip.value.locked) {
    clearHighlights()
    hideTooltip()
  }
}

const updateTooltip = (node, type) => {
  let text = ''
  
  switch (type) {
    case 'inbound':
      text = `Enrich from ${node.label} into Zeal`
      break
    case 'center':
      text = 'Fuse, enrich, brief — in seconds'
      break
    case 'outbound':
      text = `Send enriched brief to ${node.label}`
      break
  }
  
  tooltip.value.text = text
  tooltip.value.visible = true
  
  // Position tooltip (simplified - would need proper positioning logic)
  tooltip.value.x = 100
  tooltip.value.y = 50
}

const hideTooltip = () => {
  tooltip.value.visible = false
}

const highlightNode = (node, type) => {
  highlightedNodes.value = [node.id]
  
  // Highlight relevant paths
  const paths = []
  if (type === 'inbound' || type === 'center') {
    paths.push(`inbound-${node.id}`)
  }
  if (type === 'outbound' || type === 'center') {
    paths.push(`outbound-${node.id}`)
  }
  if (type === 'center') {
    // Highlight all paths when center is highlighted
    highlightedPaths.value = allPaths.value.map(p => p.id)
  } else {
    highlightedPaths.value = paths
  }
}

const clearHighlights = () => {
  highlightedNodes.value = []
  highlightedPaths.value = []
}

// Animation functions
const startPacketAnimation = () => {
  if (effectiveReducedMotion.value) return
  
  const animate = (timestamp) => {
    updatePacketPosition(timestamp)
    animationId = requestAnimationFrame(animate)
  }
  
  animationId = requestAnimationFrame(animate)
}

const stopPacketAnimation = () => {
  if (animationId) {
    cancelAnimationFrame(animationId)
    animationId = null
  }
}

const updatePacketPosition = (timestamp) => {
  if (!packetAnimation.startTime) {
    packetAnimation.startTime = timestamp
  }
  
  const elapsed = timestamp - packetAnimation.startTime
  const paths = allPaths.value
  
  if (packetAnimation.phase === 'inbound') {
    const inboundPaths = paths.filter(p => p.type === 'inbound')
    if (inboundPaths.length === 0) return
    
    const currentPath = inboundPaths[packetAnimation.currentPathIndex % inboundPaths.length]
    const progress = Math.min(elapsed / 1500, 1) // 1.5 seconds per path
    
    if (currentPath && currentPath.element) {
      const point = currentPath.element.getPointAtLength(progress * currentPath.length)
      packetPosition.value = point
      
      if (progress >= 1) {
        // Reached center, show pulse
        showPulse.value = true
        setTimeout(() => { showPulse.value = false }, 200)
        
        // Move to outbound phase
        packetAnimation.phase = 'outbound'
        packetAnimation.startTime = timestamp + props.loopDelayMs
      }
    }
  } else if (packetAnimation.phase === 'outbound') {
    const outboundPaths = paths.filter(p => p.type === 'outbound')
    if (outboundPaths.length === 0) return
    
    const currentPath = outboundPaths[packetAnimation.currentPathIndex % outboundPaths.length]
    const progress = Math.min(elapsed / 1500, 1)
    
    if (currentPath && currentPath.element) {
      const point = currentPath.element.getPointAtLength(progress * currentPath.length)
      packetPosition.value = point
      
      if (progress >= 1) {
        // Move to next cycle
        packetAnimation.currentPathIndex++
        packetAnimation.phase = 'inbound'
        packetAnimation.startTime = timestamp + props.loopDelayMs
      }
    }
  }
}

// Lifecycle
onMounted(() => {
  startPacketAnimation()
})

onBeforeUnmount(() => {
  stopPacketAnimation()
})

watch(effectiveReducedMotion, (newValue) => {
  if (newValue) {
    stopPacketAnimation()
    packetPosition.value = null
  } else {
    startPacketAnimation()
  }
})
</script>

<style scoped>
.integration-diagram {
  position: relative;
  width: 100%;
  aspect-ratio: 4.5 / 1;
  margin: 0 auto;
}

.diagram-svg {
  width: 100%;
  height: 100%;
  background: transparent;
  overflow: visible;
}

/* Paths - Modern minimalistic style */
.path-background {
  fill: none;
  stroke: rgba(166, 176, 187, 0.15);
  stroke-width: 1;
  opacity: 1;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.path-flow {
  fill: none;
  stroke: url(#flowGradient);
  stroke-width: 1.5;
  stroke-dasharray: 6 10;
  stroke-linecap: round;
  opacity: 0.8;
  animation: flow 4s linear infinite;
}

.path-flow.path-reduced-motion {
  animation: none;
  stroke-dasharray: none;
  opacity: 0.4;
}

.path-highlighted {
  stroke-width: 2;
  opacity: 1;
  filter: drop-shadow(0 0 8px rgba(46, 242, 232, 0.4));
}

.path-background.path-highlighted {
  stroke: rgba(46, 242, 232, 0.3);
  stroke-width: 1.5;
}

@keyframes flow {
  to {
    stroke-dashoffset: -16;
  }
}

/* Nodes - Modern minimalistic style */
.node-group {
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  transform-origin: center;
}

.node-group:hover,
.node-group:focus {
  outline: none;
  transform: scale(1.05);
}

.node-bg {
  stroke: rgba(255, 255, 255, 0.1);
  stroke-width: 1;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  filter: drop-shadow(0 2px 8px rgba(0, 0, 0, 0.1));
}

.node-highlighted .node-bg {
  stroke: rgba(46, 242, 232, 0.6);
  stroke-width: 2;
  filter: drop-shadow(0 0 16px rgba(46, 242, 232, 0.3));
}

.center-bg {
  stroke: rgba(46, 242, 232, 0.4);
  stroke-width: 2;
  filter: drop-shadow(0 0 20px rgba(46, 242, 232, 0.2));
}

.node-content {
  pointer-events: none;
}

.company-logo {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.node-group:hover .company-logo {
  filter: drop-shadow(0 0 8px rgba(255, 255, 255, 0.3));
}

.simple-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  color: white;
  font-weight: 600;
  font-family: system-ui, -apple-system, sans-serif;
}

.monogram-text {
  fill: white;
  font-weight: 600;
  font-family: system-ui, -apple-system, sans-serif;
}

/* Packet - Minimalistic glowing dot */
.packet {
  pointer-events: none;
  filter: drop-shadow(0 0 6px rgba(46, 242, 232, 0.8));
}

/* Pulse ring - Subtle center pulse */
.pulse-ring {
  opacity: 0.6;
  animation: pulse 0.8s ease-out;
}

@keyframes pulse {
  0% {
    r: 36;
    opacity: 0.6;
    stroke-width: 2;
  }
  100% {
    r: 60;
    opacity: 0;
    stroke-width: 1;
  }
}

/* Tooltip - Clean and minimal */
.tooltip {
  position: absolute;
  background: rgba(11, 13, 16, 0.98);
  border: 1px solid rgba(46, 242, 232, 0.2);
  border-radius: 6px;
  padding: 0.5rem 0.875rem;
  color: #e8eef2;
  font-size: 0.8rem;
  font-weight: 500;
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
  z-index: 1000;
  pointer-events: none;
  white-space: nowrap;
  transform-origin: bottom center;
  animation: tooltipIn 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes tooltipIn {
  from {
    opacity: 0;
    transform: translateY(8px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* Responsive */
@media (max-width: 768px) {
  .integration-diagram {
    max-width: 100%;
    padding: 0.5rem;
    aspect-ratio: 2.5 / 1;
  }
  
  .diagram-svg {
    viewBox: "0 0 1000 400";
  }
  
  .tooltip {
    font-size: 0.75rem;
    padding: 0.4rem 0.6rem;
    max-width: 200px;
    white-space: normal;
    text-align: center;
  }
  
  .node-group:hover,
  .node-group:focus {
    transform: scale(1.1);
  }
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  .path-flow {
    animation: none;
    stroke-dasharray: none;
  }
  
  .pulse-ring {
    animation: none;
  }
  
  .tooltip {
    animation: none;
  }
  
  .node-group,
  .path-background,
  .node-bg {
    transition: none;
  }
}
</style>

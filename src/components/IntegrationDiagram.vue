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

// Simplified logo components with better rendering
const SimpleIconLogo = {
  props: ['slug', 'size'],
  template: `
    <g class="company-logo">
      <circle r="20" fill="rgba(255,255,255,0.08)" stroke="rgba(255,255,255,0.12)" stroke-width="1"/>
      
      <!-- HubSpot -->
      <g v-if="slug === 'hubspot'">
        <circle cx="0" cy="0" r="8" fill="#FF7A59"/>
        <circle cx="0" cy="0" r="4" fill="white"/>
      </g>

      <!-- Salesforce -->
      <g v-else-if="slug === 'salesforce'">
        <path d="M-6,-3 L0,-8 L6,-3 L3,3 L-3,3 Z" fill="#00A1E0"/>
        <circle cx="0" cy="0" r="6" fill="none" stroke="#00A1E0" stroke-width="2"/>
      </g>

      <!-- Apollo -->
      <g v-else-if="slug === 'apollo'">
        <path d="M-8,6 L0,-6 L8,6 Z" fill="#6C5CE7"/>
        <circle cx="0" cy="-2" r="3" fill="#6C5CE7"/>
      </g>

      <!-- Clay -->
      <g v-else-if="slug === 'clay'">
        <rect x="-8" y="-8" width="16" height="16" rx="3" fill="#00B894"/>
        <text x="0" y="3" text-anchor="middle" fill="white" font-size="12" font-weight="bold">C</text>
      </g>

      <!-- Google -->
      <g v-else-if="slug === 'google'">
        <circle cx="0" cy="0" r="8" fill="#4285F4"/>
        <path d="M-4,-4 L4,-4 L4,0 L-4,0 Z" fill="#EA4335"/>
        <path d="M-4,0 L4,0 L4,4 L-4,4 Z" fill="#34A853"/>
      </g>

      <!-- Bing -->
      <g v-else-if="slug === 'bing'">
        <rect x="-6" y="-6" width="12" height="12" fill="#0078D4"/>
        <text x="0" y="2" text-anchor="middle" fill="white" font-size="10" font-weight="bold">b</text>
      </g>

      <!-- Gmail -->
      <g v-else-if="slug === 'gmail'">
        <rect x="-8" y="-4" width="16" height="8" rx="2" fill="#EA4335"/>
        <path d="M-8,0 L0,-4 L8,0" fill="none" stroke="white" stroke-width="2"/>
      </g>

      <!-- Slack -->
      <g v-else-if="slug === 'slack'">
        <rect x="-6" y="-2" width="4" height="4" rx="2" fill="#E01E5A"/>
        <rect x="2" y="-2" width="4" height="4" rx="2" fill="#36C5F0"/>
        <rect x="-6" y="2" width="4" height="4" rx="2" fill="#2EB67D"/>
        <rect x="2" y="2" width="4" height="4" rx="2" fill="#ECB22E"/>
      </g>

      <!-- Discord -->
      <g v-else-if="slug === 'discord'">
        <rect x="-8" y="-6" width="16" height="12" rx="4" fill="#5865F2"/>
        <circle cx="-3" cy="-1" r="1.5" fill="white"/>
        <circle cx="3" cy="-1" r="1.5" fill="white"/>
        <path d="M-4,2 Q0,4 4,2" fill="none" stroke="white" stroke-width="2"/>
      </g>

      <!-- WhatsApp -->
      <g v-else-if="slug === 'whatsapp'">
        <circle cx="0" cy="0" r="8" fill="#25D366"/>
        <path d="M-4,-2 Q0,-4 4,-2 Q4,2 0,4 Q-4,2 -4,-2" fill="white"/>
      </g>

      <!-- LinkedIn -->
      <g v-else-if="slug === 'linkedin'">
        <rect x="-8" y="-8" width="16" height="16" rx="2" fill="#0A66C2"/>
        <rect x="-5" y="-2" width="2" height="6" fill="white"/>
        <rect x="-1" y="0" width="2" height="4" fill="white"/>
        <rect x="3" y="-1" width="2" height="5" fill="white"/>
        <circle cx="-4" cy="-4" r="1" fill="white"/>
      </g>

      <!-- Globe/Web -->
      <g v-else-if="slug === 'globe'">
        <circle cx="0" cy="0" r="7" fill="none" stroke="#6B7280" stroke-width="2"/>
        <path d="M-7,0 L7,0 M0,-7 L0,7" stroke="#6B7280" stroke-width="1"/>
        <ellipse cx="0" cy="0" rx="7" ry="3" fill="none" stroke="#6B7280" stroke-width="1"/>
      </g>

      <!-- Fallback Initial Letter -->
      <g v-else>
        <circle cx="0" cy="0" r="8" fill="rgba(255,255,255,0.1)"/>
        <text x="0" y="3" text-anchor="middle" fill="white" font-size="10" font-weight="700">
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

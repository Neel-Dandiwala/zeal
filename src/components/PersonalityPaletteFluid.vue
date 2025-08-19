<template>
  <figure class="palette" :class="{ 'palette--compact': compact }">
    <div v-if="!compact" class="palette__header">
      <h3 class="palette__title">Personality Palette</h3>
      <div class="palette__controls">
        <button 
          class="palette__toggle"
          :class="{ 'palette__toggle--active': viewMode === 'big5' }"
          @click="viewMode = 'big5'"
          :aria-pressed="viewMode === 'big5'"
        >
          Big Five
        </button>
        <button 
          class="palette__toggle"
          :class="{ 'palette__toggle--active': viewMode === 'sales' }"
          @click="viewMode = 'sales'"
          :aria-pressed="viewMode === 'sales'"
        >
          Sales Traits
        </button>
      </div>
    </div>
    
    <div class="palette__canvas-container">
      <canvas 
        ref="glCanvas"
        class="palette__canvas"
        :width="canvasSize"
        :height="canvasSize"
        role="img"
        :aria-label="ariaLabel"
        @mousemove="onMouseMove"
        @mouseleave="onMouseLeave"
        @click="onCanvasClick"
        @keydown="onKeyDown"
        tabindex="0"
      ></canvas>
      
      <!-- Hidden focusable trait markers for keyboard navigation -->
      <button
        v-for="(trait, index) in processedTraits"
        :key="trait.name"
        class="palette__trait-marker visually-hidden"
        :style="getMarkerStyle(trait, index)"
        :tabindex="focusedTraitIndex === index ? 0 : -1"
        @focus="onTraitFocus(index)"
        @click="onTraitClick(index)"
        :aria-label="`${trait.name}: ${Math.round(trait.score * 100)}%`"
      >
        {{ trait.name }}
      </button>
    </div>
    
    <figcaption v-if="!compact">
      <ul class="palette__legend" role="list">
        <li v-for="(trait, index) in processedTraits" :key="trait.name" class="legend__item">
          <button 
            class="legend__button"
            @click="onLegendClick(index)"
            :aria-label="`Highlight ${trait.name} trait: ${Math.round(trait.score * 100)}%`"
          >
            <span 
              class="legend__color" 
              :style="{ backgroundColor: trait.color }"
              :aria-hidden="true"
            ></span>
            <span class="legend__name">{{ trait.name }}</span>
            <span class="legend__score">{{ Math.round(trait.score * 100) }}%</span>
          </button>
        </li>
      </ul>
    </figcaption>
    
    <!-- Tooltip -->
    <div 
      v-if="tooltip.visible"
      class="palette__tooltip"
      role="dialog"
      aria-live="polite"
      :style="tooltipStyle"
    >
      <div class="tooltip__header">
        <span class="tooltip__name">{{ tooltip.trait?.name }}</span>
        <span class="tooltip__score">{{ Math.round((tooltip.trait?.score || 0) * 100) }}%</span>
      </div>
      <div class="tooltip__bar">
        <div 
          class="tooltip__bar-fill" 
          :style="{ 
            width: `${(tooltip.trait?.score || 0) * 100}%`,
            backgroundColor: tooltip.trait?.color 
          }"
        ></div>
      </div>
      <div v-if="tooltip.trait?.source" class="tooltip__source">
        Last signal: {{ tooltip.trait.source }}
      </div>
      <button 
        class="tooltip__close"
        @click="hideTooltip"
        aria-label="Close tooltip"
      >
        ×
      </button>
    </div>
  </figure>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from 'vue'

// Configuration object for tunable parameters
const CONFIG = {
  // Spatial layout
  baseRadius: 0.55,          // Centroid radius as fraction of canvas radius
  sigmaMin: 0.12,            // Minimum RBF sigma as fraction of radius
  sigmaMax: 0.4,             // Maximum RBF sigma as fraction of radius
  sigmaRange: 0.22,          // Range for sigma calculation
  sigmaBase: 0.18,           // Base sigma offset
  
  // Color mixing
  temperature: 0.9,          // Softmax temperature
  
  // Animation
  noiseFreq: 0.08,          // Noise frequency for animation
  advectionAlpha: 0.02,     // Advection strength for sampling coords
  tetherBeta: 0.015,        // Tethering strength for centroids
  animSpeed: 0.1,           // Animation speed multiplier
  
  // Performance
  maxPixelRatio: 1.5,       // Cap pixel ratio for performance
  fallbackFPS: 25,          // Canvas 2D fallback FPS
  
  // Interaction
  pulseStrength: 0.1,       // Legend click pulse strength
  pulseDuration: 400,       // Pulse duration in ms
}

const props = defineProps({
  traits: {
    type: Array,
    default: () => []
  },
  seed: {
    type: Number,
    default: 42
  },
  animate: {
    type: Boolean,
    default: true
  },
  diameter: {
    type: Number,
    default: 520
  },
  // Legacy props for backward compatibility
  persona: {
    type: Object,
    default: null
  },
  compact: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['trait-hover', 'trait-lock'])

// Reactive refs
const glCanvas = ref(null)
const viewMode = ref('sales')
const focusedTraitIndex = ref(0)
const tooltip = ref({
  visible: false,
  trait: null,
  x: 0,
  y: 0,
  locked: false
})

// WebGL/Canvas state
let gl = null
let program = null
let uniforms = {}
let rafId = null
let resizeObserver = null
let intersectionObserver = null
let isVisible = true
let prefersReducedMotion = false
let useWebGL = false
let canvas2DContext = null
let lastRenderTime = 0

// Processed traits from props or legacy persona
const processedTraits = computed(() => {
  if (props.traits && props.traits.length > 0) {
    return [...props.traits].sort((a, b) => b.score - a.score)
  }
  
  if (props.persona && props.persona.traits) {
    return [...props.persona.traits].sort((a, b) => b.score - a.score)
  }
  
  return []
})

// Canvas size with device pixel ratio
const canvasSize = computed(() => {
  const pixelRatio = Math.min(window.devicePixelRatio || 1, CONFIG.maxPixelRatio)
  return Math.floor(props.diameter * pixelRatio)
})

// Accessibility label
const ariaLabel = computed(() => {
  const traitNames = processedTraits.value.map(t => t.name).join(', ')
  return `Personality palette visualization showing blended traits: ${traitNames}`
})

// Tooltip positioning
const tooltipStyle = computed(() => ({
  transform: `translate(${tooltip.value.x}px, ${tooltip.value.y}px)`,
  pointerEvents: tooltip.value.locked ? 'auto' : 'none'
}))

// WebGL Shaders
const vertexShaderSource = `
  attribute vec2 a_position;
  void main() {
    gl_Position = vec4(a_position, 0.0, 1.0);
  }
`

const fragmentShaderSource = `
  precision mediump float;
  
  uniform vec2 u_resolution;
  uniform float u_time;
  uniform float u_radius;
  uniform vec2 u_center;
  uniform int u_traitCount;
  uniform vec2 u_centers[16];
  uniform float u_sigmas[16];
  uniform vec3 u_colors[16];
  uniform float u_temperature;
  uniform float u_alpha;
  uniform float u_beta;
  uniform float u_seed;
  
  // Hash function for noise
  float hash21(vec2 p) {
    p = fract(p * vec2(233.34, 851.73));
    p += dot(p, p + 23.45);
    return fract(p.x * p.y);
  }
  
  // 2D Simplex noise
  float noise(vec2 p) {
    vec2 i = floor(p);
    vec2 f = fract(p);
    
    float a = hash21(i + u_seed);
    float b = hash21(i + vec2(1.0, 0.0) + u_seed);
    float c = hash21(i + vec2(0.0, 1.0) + u_seed);
    float d = hash21(i + vec2(1.0, 1.0) + u_seed);
    
    vec2 u = f * f * (3.0 - 2.0 * f);
    
    return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
  }
  
  // Curl noise (divergence-free vector field)
  vec2 curlNoise(vec2 p) {
    float eps = 0.005;
    float n1 = noise(p + vec2(0.0, eps));
    float n2 = noise(p - vec2(0.0, eps));
    float n3 = noise(p + vec2(eps, 0.0));
    float n4 = noise(p - vec2(eps, 0.0));
    
    float dx = (n1 - n2) / (2.0 * eps);
    float dy = (n3 - n4) / (2.0 * eps);
    
    return vec2(dy, -dx) * 0.3;
  }
  
  void main() {
    vec2 fragCoord = gl_FragCoord.xy;
    vec2 uv = (fragCoord - u_center) / u_radius;
    
    // Discard pixels outside circle
    float dist_from_center = length(uv);
    if (dist_from_center > 0.98) {
      discard;
    }
    
    // Apply noise-based advection for fluid motion
    vec2 noisePos = uv * 2.5 + u_seed * 0.1;
    vec2 velocity = curlNoise(noisePos + u_time * 0.4);
    vec2 samplePos = uv + velocity * u_alpha;
    
    // Start with dark base color
    vec3 finalColor = vec3(0.05, 0.06, 0.08);
    float totalWeight = 0.0;
    
    // Error check
    if (u_traitCount <= 0) {
      gl_FragColor = vec4(1.0, 0.0, 1.0, 1.0); // Magenta error
      return;
    }
    
    // Compute weighted color contributions
    for (int i = 0; i < 16; i++) {
      if (i >= u_traitCount) break;
      
      // Distance from trait center
      float dist = length(samplePos - u_centers[i]);
      
      // RBF weight calculation
      float sigma = u_sigmas[i];
      float weight = exp(-dist * dist / (2.0 * sigma * sigma));
      
      // Enhanced color contribution
      vec3 traitColor = u_colors[i];
      float colorIntensity = weight * (0.7 + 0.3 * sin(u_time * 0.5 + float(i)));
      
      finalColor += traitColor * colorIntensity;
      totalWeight += weight;
    }
    
    // Enhanced blending with saturation boost
    if (totalWeight > 0.01) {
      finalColor = finalColor / max(totalWeight, 0.1);
      
      // Boost saturation and contrast
      finalColor = mix(vec3(dot(finalColor, vec3(0.299, 0.587, 0.114))), finalColor, 1.4);
      finalColor = pow(finalColor, vec3(0.9)); // Slight gamma adjustment
    } else {
      // Use first trait color as fallback
      finalColor = u_colors[0] * 0.8;
    }
    
    // Add subtle noise for organic texture
    float grain = (hash21(fragCoord + u_time) - 0.5) * 0.03;
    finalColor += grain;
    
    // Ensure colors stay in valid range
    finalColor = clamp(finalColor, 0.0, 1.0);
    
    gl_FragColor = vec4(finalColor, 1.0);
  }
`

// Initialize WebGL
const initWebGL = () => {
  const canvas = glCanvas.value
  if (!canvas) return false
  
  // Try WebGL2 first, fallback to WebGL1
  gl = canvas.getContext('webgl2') || canvas.getContext('webgl')
  if (!gl) return false
  
  // Create shaders
  const vertexShader = createShader(gl.VERTEX_SHADER, vertexShaderSource)
  const fragmentShader = createShader(gl.FRAGMENT_SHADER, fragmentShaderSource)
  if (!vertexShader || !fragmentShader) return false
  
  // Create program
  program = gl.createProgram()
  gl.attachShader(program, vertexShader)
  gl.attachShader(program, fragmentShader)
  gl.linkProgram(program)
  
  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
    console.error('WebGL program link error:', gl.getProgramInfoLog(program))
    return false
  }
  
  gl.useProgram(program)
  
  // Set up fullscreen quad
  const positions = new Float32Array([
    -1, -1,  1, -1,  -1, 1,
    -1, 1,   1, -1,   1, 1
  ])
  
  const positionBuffer = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer)
  gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW)
  
  const positionLocation = gl.getAttribLocation(program, 'a_position')
  gl.enableVertexAttribArray(positionLocation)
  gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0)
  
  // Get uniform locations
  uniforms = {
    resolution: gl.getUniformLocation(program, 'u_resolution'),
    time: gl.getUniformLocation(program, 'u_time'),
    radius: gl.getUniformLocation(program, 'u_radius'),
    center: gl.getUniformLocation(program, 'u_center'),
    traitCount: gl.getUniformLocation(program, 'u_traitCount'),
    centers: gl.getUniformLocation(program, 'u_centers'),
    sigmas: gl.getUniformLocation(program, 'u_sigmas'),
    colors: gl.getUniformLocation(program, 'u_colors'),
    temperature: gl.getUniformLocation(program, 'u_temperature'),
    alpha: gl.getUniformLocation(program, 'u_alpha'),
    beta: gl.getUniformLocation(program, 'u_beta'),
    seed: gl.getUniformLocation(program, 'u_seed')
  }
  
  return true
}

const createShader = (type, source) => {
  const shader = gl.createShader(type)
  gl.shaderSource(shader, source)
  gl.compileShader(shader)
  
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    console.error('Shader compile error:', gl.getShaderInfoLog(shader))
    gl.deleteShader(shader)
    return null
  }
  
  return shader
}

// Initialize Canvas 2D fallback
const initCanvas2D = () => {
  const canvas = glCanvas.value
  if (!canvas) return false
  
  canvas2DContext = canvas.getContext('2d')
  return !!canvas2DContext
}

// Convert hex color to RGB array
const hexToRgb = (hex) => {
  if (!hex || typeof hex !== 'string') {
    console.warn('Invalid hex color:', hex)
    return [1, 0, 1] // Magenta as error indicator
  }
  
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
  if (!result) {
    console.warn('Failed to parse hex color:', hex)
    return [1, 0, 1] // Magenta as error indicator
  }
  
  const rgb = [
    parseInt(result[1], 16) / 255,
    parseInt(result[2], 16) / 255,
    parseInt(result[3], 16) / 255
  ]
  
  console.log(`Color ${hex} -> RGB:`, rgb)
  return rgb
}

// Set up trait layout and uniforms
const setupUniforms = () => {
  const canvas = glCanvas.value
  if (!canvas) {
    console.warn('Canvas not available for setupUniforms')
    return
  }
  
  const traits = processedTraits.value
  if (traits.length === 0) {
    console.warn('No traits available for setupUniforms')
    return
  }
  
  console.log('Setting up uniforms for traits:', traits)
  
  const radius = canvas.width / 2
  const center = [radius, radius]
  
  // Calculate sector angles based on scores
  const totalScore = traits.reduce((sum, trait) => sum + trait.score, 0)
  let currentAngle = 0
  const centers = []
  const sigmas = []
  const colors = []
  
  traits.forEach((trait, index) => {
    const sectorAngle = (trait.score / totalScore) * 2 * Math.PI
    const bisectorAngle = currentAngle + sectorAngle / 2
    
    // Centroid position
    const centroidRadius = CONFIG.baseRadius * radius
    const centroidX = Math.cos(bisectorAngle) * centroidRadius
    const centroidY = Math.sin(bisectorAngle) * centroidRadius
    
    // Normalize to UV coordinates (-1 to 1)
    centers.push(centroidX / radius, centroidY / radius)
    
    // Calculate sigma based on score
    const sigmaScale = CONFIG.sigmaBase + CONFIG.sigmaRange * (1 - trait.score)
    const sigma = Math.max(CONFIG.sigmaMin, Math.min(CONFIG.sigmaMax, sigmaScale))
    sigmas.push(sigma)
    
    // Convert color to linear RGB
    const rgb = hexToRgb(trait.color)
    colors.push(...rgb)
    
    currentAngle += sectorAngle
    
    console.log(`Trait ${index}: ${trait.name}`, {
      color: trait.color,
      rgb: rgb,
      score: trait.score,
      sigma: sigma,
      centroid: [centroidX / radius, centroidY / radius]
    })
  })
  
  console.log('Final colors array:', colors)
  console.log('Final centers array:', centers)
  console.log('Final sigmas array:', sigmas)
  
  if (useWebGL && gl && program) {
    console.log('Setting WebGL uniforms...')
    gl.uniform2f(uniforms.resolution, canvas.width, canvas.height)
    gl.uniform1f(uniforms.radius, radius)
    gl.uniform2f(uniforms.center, center[0], center[1])
    gl.uniform1i(uniforms.traitCount, traits.length)
    gl.uniform2fv(uniforms.centers, new Float32Array(centers))
    gl.uniform1fv(uniforms.sigmas, new Float32Array(sigmas))
    gl.uniform3fv(uniforms.colors, new Float32Array(colors))
    gl.uniform1f(uniforms.temperature, CONFIG.temperature)
    gl.uniform1f(uniforms.alpha, CONFIG.advectionAlpha)
    gl.uniform1f(uniforms.beta, CONFIG.tetherBeta)
    gl.uniform1f(uniforms.seed, props.seed)
    console.log('WebGL uniforms set successfully')
  } else {
    console.log('WebGL not available, using Canvas 2D fallback')
  }
}

// Render frame
const render = (time) => {
  const canvas = glCanvas.value
  if (!canvas) return
  
  if (useWebGL && gl && program) {
    // WebGL rendering
    gl.viewport(0, 0, canvas.width, canvas.height)
    gl.uniform1f(uniforms.time, time * CONFIG.animSpeed)
    gl.drawArrays(gl.TRIANGLES, 0, 6)
  } else if (canvas2DContext) {
    // Canvas 2D fallback rendering
    renderCanvas2D(time)
  }
}

// Canvas 2D fallback implementation
const renderCanvas2D = (time) => {
  const canvas = glCanvas.value
  const ctx = canvas2DContext
  if (!canvas || !ctx) return
  
  const traits = processedTraits.value
  if (traits.length === 0) return
  
  const size = canvas.width
  const radius = size / 2
  
  // Clear canvas with very dark background
  ctx.fillStyle = '#050608'
  ctx.fillRect(0, 0, size, size)
  
  // Create circular clipping path
  ctx.save()
  ctx.beginPath()
  ctx.arc(radius, radius, radius * 0.98, 0, 2 * Math.PI)
  ctx.clip()
  
  // Fill circle with dark base
  ctx.fillStyle = '#0a0c0f'
  ctx.beginPath()
  ctx.arc(radius, radius, radius * 0.98, 0, 2 * Math.PI)
  ctx.fill()
  
  // Calculate total score for proportional sectors
  const totalScore = traits.reduce((sum, trait) => sum + trait.score, 0)
  let currentAngle = 0
  
  // Animation offset
  const animOffset = props.animate && !prefersReducedMotion ? time * 0.08 : 0
  
  // Draw multiple layers for richer blending
  for (let layer = 0; layer < 3; layer++) {
    currentAngle = 0
    
    traits.forEach((trait, index) => {
      const sectorAngle = (trait.score / totalScore) * 2 * Math.PI
      const centerAngle = currentAngle + sectorAngle / 2 + animOffset * (layer + 1) * 0.5
      
      // Multiple influence points for organic mixing
      const numInfluences = layer === 0 ? 1 : 2
      
      for (let inf = 0; inf < numInfluences; inf++) {
        const angleOffset = inf * Math.PI / 3
        const influenceRadius = radius * (CONFIG.baseRadius + layer * 0.15)
        const centerX = radius + Math.cos(centerAngle + angleOffset) * influenceRadius
        const centerY = radius + Math.sin(centerAngle + angleOffset) * influenceRadius
        
        // Large overlapping gradients
        const gradientRadius = radius * (0.9 + trait.score * 0.8 + layer * 0.2)
        const gradient = ctx.createRadialGradient(
          centerX, centerY, 0,
          centerX, centerY, gradientRadius
        )
        
        // High saturation colors
        const baseOpacity = layer === 0 ? 0.9 : 0.6
        const opacity = Math.max(baseOpacity, trait.score * 0.95)
        const opacityHex = Math.floor(opacity * 255).toString(16).padStart(2, '0')
        
        gradient.addColorStop(0, trait.color + opacityHex)
        gradient.addColorStop(0.4, trait.color + Math.floor(opacity * 0.7 * 255).toString(16).padStart(2, '0'))
        gradient.addColorStop(0.8, trait.color + Math.floor(opacity * 0.3 * 255).toString(16).padStart(2, '0'))
        gradient.addColorStop(1, trait.color + '00')
        
        // Advanced blending modes
        if (index === 0 && layer === 0 && inf === 0) {
          ctx.globalCompositeOperation = 'source-over'
        } else if (layer === 0) {
          ctx.globalCompositeOperation = 'screen'
        } else if (layer === 1) {
          ctx.globalCompositeOperation = 'multiply'
        } else {
          ctx.globalCompositeOperation = 'overlay'
        }
        
        ctx.fillStyle = gradient
        ctx.beginPath()
        ctx.arc(centerX, centerY, gradientRadius, 0, 2 * Math.PI)
        ctx.fill()
      }
      
      currentAngle += sectorAngle
    })
  }
  
  // Final enhancement layer
  ctx.globalCompositeOperation = 'soft-light'
  const enhanceGradient = ctx.createRadialGradient(
    radius, radius, 0,
    radius, radius, radius * 0.9
  )
  enhanceGradient.addColorStop(0, 'rgba(91, 215, 194, 0.2)')
  enhanceGradient.addColorStop(0.5, 'rgba(122, 162, 255, 0.15)')
  enhanceGradient.addColorStop(1, 'rgba(0, 0, 0, 0.1)')
  ctx.fillStyle = enhanceGradient
  ctx.beginPath()
  ctx.arc(radius, radius, radius * 0.9, 0, 2 * Math.PI)
  ctx.fill()
  
  ctx.restore()
}

// Animation loop
const startAnimation = () => {
  if (!props.animate || prefersReducedMotion || !isVisible) {
    render(0)
    return
  }
  
  const loop = (timestamp) => {
    const time = timestamp * 0.001 // Convert to seconds
    
    // Throttle to target FPS
    if (timestamp - lastRenderTime >= (1000 / (useWebGL ? 60 : CONFIG.fallbackFPS))) {
      render(time)
      lastRenderTime = timestamp
    }
    
    rafId = requestAnimationFrame(loop)
  }
  
  rafId = requestAnimationFrame(loop)
}

const stopAnimation = () => {
  if (rafId) {
    cancelAnimationFrame(rafId)
    rafId = null
  }
}

// Mouse interaction handlers
const onMouseMove = (event) => {
  if (!tooltip.value.locked) {
    updateTooltip(event)
  }
}

const onMouseLeave = () => {
  if (!tooltip.value.locked) {
    hideTooltip()
  }
}

const onCanvasClick = (event) => {
  const trait = getDominantTrait(event)
  if (trait) {
    tooltip.value.locked = !tooltip.value.locked
    if (tooltip.value.locked) {
      updateTooltip(event)
      emit('trait-lock', trait)
    } else {
      hideTooltip()
    }
  }
}

// Keyboard navigation
const onKeyDown = (event) => {
  const traits = processedTraits.value
  let newIndex = focusedTraitIndex.value
  
  switch (event.key) {
    case 'ArrowRight':
      newIndex = (newIndex + 1) % traits.length
      break
    case 'ArrowLeft':
      newIndex = newIndex === 0 ? traits.length - 1 : newIndex - 1
      break
    case 'Enter':
    case ' ':
      event.preventDefault()
      showTraitTooltip(newIndex, event)
      return
    case 'Escape':
      hideTooltip()
      return
    default:
      return
  }
  
  event.preventDefault()
  focusedTraitIndex.value = newIndex
  
  // Focus the trait marker
  const marker = event.currentTarget.parentElement.querySelector(
    `.palette__trait-marker:nth-child(${newIndex + 2})`
  )
  marker?.focus()
}

// Get dominant trait at cursor position
const getDominantTrait = (event) => {
  const canvas = glCanvas.value
  if (!canvas) return null
  
  const rect = canvas.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  
  // Convert to canvas coordinates
  const canvasX = (x / rect.width) * canvas.width
  const canvasY = (y / rect.height) * canvas.height
  
  // Convert to UV coordinates
  const centerX = canvas.width / 2
  const centerY = canvas.height / 2
  const radius = canvas.width / 2
  
  const uvX = (canvasX - centerX) / radius
  const uvY = (canvasY - centerY) / radius
  
  // Check if inside circle
  if (Math.sqrt(uvX * uvX + uvY * uvY) > 1) return null
  
  // Find closest trait centroid (simplified)
  const traits = processedTraits.value
  let closestTrait = null
  let minDistance = Infinity
  
  traits.forEach((trait, index) => {
    const angle = (index / traits.length) * 2 * Math.PI
    const centroidX = Math.cos(angle) * CONFIG.baseRadius
    const centroidY = Math.sin(angle) * CONFIG.baseRadius
    
    const distance = Math.sqrt(
      (uvX - centroidX) ** 2 + (uvY - centroidY) ** 2
    )
    
    if (distance < minDistance) {
      minDistance = distance
      closestTrait = trait
    }
  })
  
  return closestTrait
}

// Update tooltip position and content
const updateTooltip = (event) => {
  const trait = getDominantTrait(event)
  if (!trait) return
  
  const rect = glCanvas.value.getBoundingClientRect()
  tooltip.value = {
    visible: true,
    trait,
    x: event.clientX - rect.left + 10,
    y: event.clientY - rect.top - 10,
    locked: tooltip.value.locked
  }
  
  emit('trait-hover', trait)
}

const hideTooltip = () => {
  tooltip.value = {
    visible: false,
    trait: null,
    x: 0,
    y: 0,
    locked: false
  }
}

const showTraitTooltip = (index, event) => {
  const trait = processedTraits.value[index]
  if (trait) {
    tooltip.value = {
      visible: true,
      trait,
      x: 200,
      y: 100,
      locked: true
    }
    emit('trait-hover', trait)
  }
}

// Legend interactions
const onLegendClick = (index) => {
  const trait = processedTraits.value[index]
  if (trait) {
    // Pulse effect - briefly increase trait prominence
    emit('trait-hover', trait)
  }
}

// Trait marker positioning
const getMarkerStyle = (trait, index) => {
  // Position markers over trait centroids
  const angle = (index / processedTraits.value.length) * 2 * Math.PI
  const radius = props.diameter / 2 * CONFIG.baseRadius
  const x = Math.cos(angle) * radius + props.diameter / 2
  const y = Math.sin(angle) * radius + props.diameter / 2
  
  return {
    position: 'absolute',
    left: `${x}px`,
    top: `${y}px`,
    transform: 'translate(-50%, -50%)'
  }
}

// Focus handlers for trait markers
const onTraitFocus = (index) => {
  focusedTraitIndex.value = index
  showTraitTooltip(index, { clientX: 0, clientY: 0 })
}

const onTraitClick = (index) => {
  const trait = processedTraits.value[index]
  if (trait) {
    emit('trait-lock', trait)
  }
}

// Expose methods
const capturePNG = () => {
  const canvas = glCanvas.value
  if (!canvas) return null
  
  return canvas.toDataURL('image/png')
}

// Lifecycle hooks
onMounted(() => {
  console.log('PersonalityPaletteFluid mounted')
  
  // Check for reduced motion preference
  prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  console.log('Prefers reduced motion:', prefersReducedMotion)
  
  // Try WebGL first, fallback to Canvas 2D
  if (initWebGL()) {
    useWebGL = true
    console.log('Using WebGL rendering')
  } else if (initCanvas2D()) {
    useWebGL = false
    console.log('Using Canvas 2D rendering (WebGL fallback)')
  } else {
    console.error('Neither WebGL nor Canvas 2D is available')
    return
  }
  
  console.log('Canvas size:', canvasSize.value)
  console.log('Processed traits:', processedTraits.value)
  
  setupUniforms()
  startAnimation()
  
  // Set up resize observer
  if (window.ResizeObserver) {
    resizeObserver = new ResizeObserver(() => {
      console.log('Canvas resized, updating uniforms')
      setupUniforms()
    })
    resizeObserver.observe(glCanvas.value)
  }
  
  // Set up intersection observer for performance
  if (window.IntersectionObserver) {
    intersectionObserver = new IntersectionObserver((entries) => {
      isVisible = entries[0].isIntersecting
      console.log('Visibility changed:', isVisible)
      if (isVisible) {
        startAnimation()
      } else {
        stopAnimation()
      }
    })
    intersectionObserver.observe(glCanvas.value)
  }
})

onBeforeUnmount(() => {
  stopAnimation()
  resizeObserver?.disconnect()
  intersectionObserver?.disconnect()
  
  if (gl) {
    gl.deleteProgram(program)
    program = null
    gl = null
  }
})

// Watch for trait changes
watch(() => processedTraits.value, () => {
  setupUniforms()
  if (!props.animate || prefersReducedMotion) {
    render(0)
  }
}, { deep: true })

watch(() => props.animate, (newValue) => {
  if (newValue && !prefersReducedMotion && isVisible) {
    startAnimation()
  } else {
    stopAnimation()
    render(0)
  }
})

// Expose public API
defineExpose({
  capturePNG
})
</script>

<style scoped>
.palette {
  background: var(--panel);
  border-radius: var(--border-radius-lg);
  padding: 2rem;
  border: 1px solid rgba(166, 176, 187, 0.1);
  position: relative;
}

.palette--compact {
  padding: 1rem;
}

.palette__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.palette__title {
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
}

.palette__controls {
  display: flex;
  gap: 0.5rem;
}

.palette__toggle {
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

.palette__toggle:hover,
.palette__toggle:focus {
  border-color: var(--accent);
  color: var(--text);
}

.palette__toggle--active {
  background: var(--accent);
  color: var(--bg);
  border-color: var(--accent);
}

.palette__canvas-container {
  position: relative;
  display: flex;
  justify-content: center;
  margin-bottom: 1.5rem;
}

.palette__canvas {
  display: block;
  border-radius: 50%;
  border: 2px solid rgba(166, 176, 187, 0.1);
  transition: var(--transition);
  cursor: pointer;
  outline: none;
  background: transparent;
}

.palette__canvas:hover {
  border-color: var(--accent);
  box-shadow: 0 0 20px rgba(91, 215, 194, 0.3);
}

.palette__canvas:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(91, 215, 194, 0.3);
}

.palette__trait-marker {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

.palette__trait-marker:focus {
  position: static;
  width: auto;
  height: auto;
  overflow: visible;
  clip: auto;
  white-space: normal;
  padding: 0.5rem;
  background: var(--accent);
  color: var(--bg);
  border-radius: var(--border-radius);
}

.palette__legend {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  justify-content: center;
}

.legend__item {
  margin: 0;
}

.legend__button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 0.75rem;
  background: transparent;
  border: 1px solid rgba(166, 176, 187, 0.2);
  border-radius: var(--border-radius);
  color: var(--text);
  font-family: inherit;
  font-size: 0.875rem;
  cursor: pointer;
  transition: var(--transition);
}

.legend__button:hover,
.legend__button:focus {
  border-color: var(--accent);
  background: rgba(91, 215, 194, 0.1);
  transform: translateY(-1px);
}

.legend__color {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  flex-shrink: 0;
}

.legend__name {
  font-weight: 500;
}

.legend__score {
  color: var(--muted);
  font-size: 0.75rem;
}

.palette__tooltip {
  position: absolute;
  background: var(--bg);
  border: 1px solid rgba(166, 176, 187, 0.3);
  border-radius: var(--border-radius);
  padding: 1rem;
  min-width: 200px;
  box-shadow: var(--shadow-lg);
  z-index: 10;
  pointer-events: none;
}

.tooltip__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.tooltip__name {
  font-weight: 600;
  color: var(--text);
}

.tooltip__score {
  font-weight: 700;
  color: var(--accent);
}

.tooltip__bar {
  height: 4px;
  background: rgba(166, 176, 187, 0.2);
  border-radius: 2px;
  overflow: hidden;
  margin-bottom: 0.75rem;
}

.tooltip__bar-fill {
  height: 100%;
  transition: var(--transition);
}

.tooltip__source {
  font-size: 0.75rem;
  color: var(--muted);
  font-style: italic;
}

.tooltip__close {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  width: 24px;
  height: 24px;
  background: none;
  border: none;
  color: var(--muted);
  font-size: 18px;
  line-height: 1;
  cursor: pointer;
  transition: var(--transition);
}

.tooltip__close:hover {
  color: var(--text);
}

@media (max-width: 768px) {
  .palette {
    padding: 1.5rem;
  }
  
  .palette__header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .palette__legend {
    gap: 0.5rem;
  }
  
  .legend__button {
    padding: 0.375rem 0.5rem;
    font-size: 0.75rem;
  }
  
  .palette__tooltip {
    position: fixed;
    top: 50%;
    left: 1rem;
    right: 1rem;
    transform: translateY(-50%);
    min-width: auto;
  }
}

@media (prefers-reduced-motion: reduce) {
  .palette__canvas,
  .legend__button,
  .tooltip__bar-fill {
    transition: none;
  }
  
  .legend__button:hover {
    transform: none;
  }
}
</style>

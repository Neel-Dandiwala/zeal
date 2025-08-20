<template>
  <div class="personality-palette-nebula">
    <div class="palette-container">
      <!-- Three.js Canvas with CSS Fallback -->
      <div class="canvas-container">
        <canvas
          ref="canvasRef"
          class="nebula-canvas"
          :width="canvasSize"
          :height="canvasSize"
          :style="{ 
            width: canvasSize + 'px', 
            height: canvasSize + 'px',
            display: 'block'
          }"
          @click="handleCanvasClick"
          @mousemove="handleMouseMove"
          @mouseleave="handleMouseLeave"
        />
        
        <!-- CSS Fallback Circle -->
        <div 
          v-if="!threeJsWorking"
          class="css-fallback-circle"
          :style="{ 
            width: canvasSize + 'px', 
            height: canvasSize + 'px'
          }"
        >
          <div class="nebula-gradient"></div>
          <div class="grain-overlay"></div>
        </div>
      </div>
      
      <!-- Trait Legend -->
      <div class="trait-legend">
        <div
          v-for="(trait, key, index) in traits"
          :key="key"
          class="trait-pill"
          :class="{ 'trait-pill--highlighted': highlightedTrait === key }"
          @mouseenter="highlightTrait(key)"
          @mouseleave="clearHighlight"
        >
          <div 
            class="trait-color-ring"
            :style="{ 
              backgroundColor: getToneMappedColor(trait.color),
              boxShadow: `0 0 16px ${getToneMappedColor(trait.color)}40`
            }"
          >
            <div class="trait-inner-circle">
              <span class="trait-label">{{ formatTraitName(key) }}</span>
              <span class="trait-percentage">{{ Math.round(trait.weight * 100) }}%</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Tooltip -->
    <div
      v-if="tooltip.visible"
      class="palette-tooltip"
      :style="{ 
        left: tooltip.x + 'px', 
        top: tooltip.y + 'px',
        backgroundColor: tooltip.dominantColor + '20',
        borderColor: tooltip.dominantColor + '60'
      }"
    >
      <div class="tooltip-trait">{{ tooltip.traitName }}</div>
      <div class="tooltip-strength">{{ tooltip.strength }}% influence</div>
    </div>
    

  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch, computed } from 'vue'
import * as THREE from 'three'

// Props
const props = defineProps({
  traits: {
    type: Object,
    required: true,
    validator: (traits) => {
      const requiredKeys = ['mind', 'energy', 'nature', 'tactics', 'identity']
      return requiredKeys.every(key => 
        traits[key] && 
        typeof traits[key].color === 'string' && 
        typeof traits[key].weight === 'number'
      )
    }
  },
  size: {
    type: Number,
    default: 400
  },
  animate: {
    type: Boolean,
    default: true
  }
})

// Reactive state
const canvasRef = ref(null)
const highlightedTrait = ref(null)
const tooltip = ref({
  visible: false,
  x: 0,
  y: 0,
  traitName: '',
  strength: 0,
  dominantColor: '#2EF2E8'
})

const threeJsWorking = ref(false)

// Three.js objects
let scene, camera, renderer, uniforms, material, mesh
let animationId = null
let startTime = Date.now()

// Computed properties
const canvasSize = computed(() => props.size)

// Trait color processing
const hexToRgb = (hex) => {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
  if (!result) return [1, 1, 1]
  return [
    parseInt(result[1], 16) / 255,
    parseInt(result[2], 16) / 255,
    parseInt(result[3], 16) / 255
  ]
}

const rgbToHsv = (r, g, b) => {
  const max = Math.max(r, g, b)
  const min = Math.min(r, g, b)
  const delta = max - min
  
  let h = 0
  if (delta !== 0) {
    if (max === r) h = ((g - b) / delta) % 6
    else if (max === g) h = (b - r) / delta + 2
    else h = (r - g) / delta + 4
  }
  h = Math.round(h * 60)
  if (h < 0) h += 360
  
  const s = max === 0 ? 0 : delta / max
  const v = max
  
  return [h / 360, s, v]
}

const hsvToRgb = (h, s, v) => {
  h *= 360
  const c = v * s
  const x = c * (1 - Math.abs((h / 60) % 2 - 1))
  const m = v - c
  
  let r, g, b
  if (h >= 0 && h < 60) [r, g, b] = [c, x, 0]
  else if (h >= 60 && h < 120) [r, g, b] = [x, c, 0]
  else if (h >= 120 && h < 180) [r, g, b] = [0, c, x]
  else if (h >= 180 && h < 240) [r, g, b] = [0, x, c]
  else if (h >= 240 && h < 300) [r, g, b] = [x, 0, c]
  else [r, g, b] = [c, 0, x]
  
  return [r + m, g + m, b + m]
}

const getToneMappedColor = (hex) => {
  const [r, g, b] = hexToRgb(hex)
  const [h, s, v] = rgbToHsv(r, g, b)
  
  // Apply same tone mapping as shader
  const enhancedS = s * 1.12
  const clampedV = Math.min(v, 0.94)
  
  const [newR, newG, newB] = hsvToRgb(h, enhancedS, clampedV)
  
  return `rgb(${Math.round(newR * 255)}, ${Math.round(newG * 255)}, ${Math.round(newB * 255)})`
}

const formatTraitName = (key) => {
  const names = {
    mind: 'Mind',
    energy: 'Energy', 
    nature: 'Nature',
    tactics: 'Tactics',
    identity: 'Identity'
  }
  return names[key] || key
}

// Vertex shader - Three.js compatible
const vertexShader = `
varying vec2 vUv;

void main() {
  vUv = uv;
  gl_Position = vec4(position, 1.0);
}
`

// Fragment shader - Full nebula effect
const fragmentShader = `
precision highp float;

varying vec2 vUv;
uniform float uTime;
uniform float uRadius;

// Hash function for procedural noise
float hash(vec2 p) {
  p = fract(p * vec2(123.34, 345.45));
  p += dot(p, p + 34.345);
  return fract(p.x * p.y);
}

// Simple noise function
float noise(vec2 p) {
  vec2 i = floor(p);
  vec2 f = fract(p);
  float a = hash(i);
  float b = hash(i + vec2(1.0, 0.0));
  float c = hash(i + vec2(0.0, 1.0));
  float d = hash(i + vec2(1.0, 1.0));
  vec2 u = f * f * (3.0 - 2.0 * f);
  return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}

// Fractal noise (simplified)
float fbm(vec2 p) {
  float value = 0.0;
  float amplitude = 0.5;
  for (int i = 0; i < 4; i++) {
    value += amplitude * noise(p);
    p *= 2.0;
    amplitude *= 0.5;
  }
  return value;
}

// HSV color space conversion
vec3 hsv2rgb(vec3 c) {
  vec4 K = vec4(1.0, 2.0 / 3.0, 1.0 / 3.0, 3.0);
  vec3 p = abs(fract(c.xxx + K.xyz) * 6.0 - K.www);
  return c.z * mix(K.xxx, clamp(p - K.xxx, 0.0, 1.0), c.y);
}

void main() {
  vec2 uv = vUv - 0.5;
  float r = length(uv);
  
  if (r > uRadius) {
    discard;
  }
  
  // Create organic flowing noise field
  vec2 p = uv * 3.0;
  float n1 = fbm(p + uTime * 0.1);
  float n2 = fbm(p * 1.3 + vec2(2.0, -1.0) + uTime * 0.07);
  vec2 warp = vec2(n1, n2) - 0.5;
  vec2 q = p + warp * 1.5;
  
  // Combined noise field for color blending
  float field = fbm(q) * 0.8 + fbm(q * 2.1) * 0.2;
  field = clamp(field, 0.0, 1.0);
  
  // Reference nebula colors (matching your image)
  vec3 colors[5];
  colors[0] = vec3(0.95, 0.35, 0.15);  // Deep red-orange
  colors[1] = vec3(0.15, 0.85, 0.95);  // Bright cyan-teal
  colors[2] = vec3(0.65, 0.25, 0.85);  // Deep purple-magenta
  colors[3] = vec3(0.45, 0.35, 0.95);  // Blue-violet
  colors[4] = vec3(0.25, 0.65, 0.75);  // Darker teal-blue
  
  // Blend colors based on noise field
  vec3 color;
  if (field < 0.2) {
    color = mix(colors[0], colors[1], field * 5.0);
  } else if (field < 0.4) {
    color = mix(colors[1], colors[2], (field - 0.2) * 5.0);
  } else if (field < 0.6) {
    color = mix(colors[2], colors[3], (field - 0.4) * 5.0);
  } else if (field < 0.8) {
    color = mix(colors[3], colors[4], (field - 0.6) * 5.0);
  } else {
    color = mix(colors[4], colors[0], (field - 0.8) * 5.0);
  }
  
  // Boost saturation for richer colors
  float saturation = 1.3;
  float gray = dot(color, vec3(0.299, 0.587, 0.114));
  color = mix(vec3(gray), color, saturation);
  
  // Darken overall for rich tones
  color *= 0.8;
  
  // Vignette effect for depth
  float vignette = smoothstep(uRadius * 0.9, uRadius * 0.6, r);
  color *= mix(1.2, 0.7, vignette);
  
  // Add grain texture
  vec2 fragCoord = gl_FragCoord.xy;
  float grain1 = fract(sin(dot(fragCoord, vec2(12.9898, 78.233))) * 43758.5453);
  float grain2 = noise(q * 8.0 + 13.7);
  float speckle = smoothstep(0.4, 0.9, grain2 + grain1 * 0.4);
  
  // Apply grain for stipple effect
  color = mix(color * 0.85, color * 1.15, speckle);
  
  // Prevent pure whites
  float maxChannel = max(max(color.r, color.g), color.b);
  if (maxChannel > 0.9) {
    color *= 0.9 / maxChannel;
  }
  
  gl_FragColor = vec4(color, 1.0);
}
`

// Ensure perfect circular canvas sizing
const resizeCanvas = () => {
  if (!canvasRef.value || !renderer) return
  
  const dpr = Math.min(window.devicePixelRatio || 1, 1.5)
  const { width, height } = canvasRef.value.getBoundingClientRect()
  const size = Math.min(width, height)
  
  // Set canvas size to perfect square
  canvasRef.value.width = size * dpr
  canvasRef.value.height = size * dpr
  
  // Update renderer size
  renderer.setSize(size, size)
  renderer.setPixelRatio(dpr)
  
  // Update viewport
  if (renderer.getContext) {
    const gl = renderer.getContext()
    gl.viewport(0, 0, canvasRef.value.width, canvasRef.value.height)
  }
  
  // Update uniforms
  if (uniforms && uniforms.uResolution) {
    uniforms.uResolution.value.set(size, size)
  }
}

// Initialize Three.js scene
const initThreeJS = () => {
  if (!canvasRef.value) {
    console.error('Canvas ref not available')
    return
  }
  
  console.log('Initializing Three.js scene...')
  
  // Scene setup
  scene = new THREE.Scene()
  camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0.1, 10)
  camera.position.z = 1
  
  // Renderer
  renderer = new THREE.WebGLRenderer({ 
    canvas: canvasRef.value,
    alpha: true,
    antialias: false,
    preserveDrawingBuffer: true
  })
  
  // Set color space explicitly
  renderer.outputColorSpace = THREE.SRGBColorSpace
  
  // Perfect circle sizing
  resizeCanvas()
  renderer.setClearColor(0x000000, 0) // Transparent background
  
  console.log('Renderer created:', renderer)
  
  // Shader uniforms - simplified for testing
  uniforms = {
    uTime: { value: 0 },
    uResolution: { value: new THREE.Vector2(canvasSize.value, canvasSize.value) },
    uTraitColors: { value: [] },
    uTraitWeights: { value: [] },
    uRadius: { value: 0.48 }
  }
  
  // Update uniforms from props
  updateUniforms()
  
  console.log('Uniforms:', uniforms)
  
  // Remove test circle and go straight to shader
  threeJsWorking.value = true
  
  // Now try shader material with error handling
  try {
    material = new THREE.ShaderMaterial({
      vertexShader,
      fragmentShader,
      uniforms,
      transparent: true,
      side: THREE.DoubleSide
    })
    
    console.log('Shader material created successfully:', material)
    
    // Check for shader compilation errors
    const gl = renderer.getContext()
    renderer.compile(scene, camera)
    
    if (material.program && material.program.diagnostics) {
      console.log('Shader diagnostics:', material.program.diagnostics)
    }
    
  } catch (error) {
    console.error('Shader compilation failed:', error)
    // Fallback to simple colored material
    material = new THREE.MeshBasicMaterial({ 
      color: 0x2EF2E8,
      transparent: true,
      opacity: 0.8
    })
    console.log('Using fallback material')
  }
  
  // Full-screen quad
  const geometry = new THREE.PlaneGeometry(2, 2)
  mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)
  
  console.log('Mesh added to scene:', mesh)
  
  // Test render first
  renderer.render(scene, camera)
  console.log('Initial render complete')
  
  // Start animation loop
  if (props.animate) {
    animate()
  } else {
    render()
  }
}

// Update shader uniforms from Vue props
const updateUniforms = () => {
  if (!uniforms) return
  
  const traitKeys = ['mind', 'energy', 'nature', 'tactics', 'identity']
  const colors = []
  const weights = []
  
  console.log('Updating uniforms with traits:', props.traits)
  
  traitKeys.forEach(key => {
    const trait = props.traits[key]
    if (trait) {
      const rgb = hexToRgb(trait.color)
      console.log(`Trait ${key}: color=${trait.color}, rgb=[${rgb[0]}, ${rgb[1]}, ${rgb[2]}], weight=${trait.weight}`)
      colors.push(new THREE.Vector3(...rgb))
      weights.push(trait.weight)
    } else {
      console.warn(`Missing trait: ${key}`)
      colors.push(new THREE.Vector3(1, 0, 0)) // Red fallback
      weights.push(0.2)
    }
  })
  
  // Force update the uniform values
  uniforms.uTraitColors.value = colors
  uniforms.uTraitWeights.value = weights
  
  // Mark uniforms as needing update
  if (material) {
    material.uniformsNeedUpdate = true
    
    // Also manually update the uniforms if material is compiled
    if (material.program) {
      material.uniforms.uTraitColors.value = colors
      material.uniforms.uTraitWeights.value = weights
    }
  }
  
  console.log('Final colors:', colors)
  console.log('Final weights:', weights)
  console.log('Material updated:', material ? 'Yes' : 'No')
}

// Animation state
let isVisible = true
let reducedMotion = false

// Check for reduced motion preference
const checkReducedMotion = () => {
  reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
}

// Animation loop
const animate = () => {
  if (!props.animate || reducedMotion || !isVisible) {
    // Render single static frame for reduced motion or when not visible
    render()
    return
  }
  
  const currentTime = Date.now()
  uniforms.uTime.value = (currentTime - startTime) * 0.001
  
  render()
  animationId = requestAnimationFrame(animate)
}

// Render frame
const render = () => {
  if (renderer && scene && camera) {
    renderer.render(scene, camera)
  }
}

// Event handlers
const handleCanvasClick = (event) => {
  // Could add click-to-lock tooltip functionality
}

const handleMouseMove = (event) => {
  const rect = canvasRef.value.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  
  // Convert to UV coordinates
  const uv = {
    x: x / rect.width,
    y: 1 - (y / rect.height)
  }
  
  // Check if inside circle and update tooltip
  const centerX = 0.5
  const centerY = 0.5
  const dx = uv.x - centerX
  const dy = uv.y - centerY
  const distance = Math.sqrt(dx * dx + dy * dy)
  
  if (distance < 0.48) {
    // Sample dominant trait at this position (simplified)
    const angle = Math.atan2(dy, dx) + Math.PI
    const normalizedAngle = angle / (2 * Math.PI)
    
    const traitKeys = ['mind', 'energy', 'nature', 'tactics', 'identity']
    const traitIndex = Math.floor(normalizedAngle * traitKeys.length)
    const traitKey = traitKeys[traitIndex]
    const trait = props.traits[traitKey]
    
    tooltip.value = {
      visible: true,
      x: event.clientX + 10,
      y: event.clientY - 40,
      traitName: formatTraitName(traitKey),
      strength: Math.round(trait.weight * 100),
      dominantColor: getToneMappedColor(trait.color)
    }
  } else {
    tooltip.value.visible = false
  }
}

const handleMouseLeave = () => {
  tooltip.value.visible = false
}

const highlightTrait = (traitKey) => {
  highlightedTrait.value = traitKey
}

const clearHighlight = () => {
  highlightedTrait.value = null
}

// Debug functions
const debugRender = () => {
  console.log('Force rendering...')
  if (renderer && scene && camera) {
    updateUniforms()
    
    // Try recreating the material with fresh uniforms
    try {
      const freshMaterial = new THREE.ShaderMaterial({
        vertexShader,
        fragmentShader,
        uniforms,
        transparent: true,
        side: THREE.DoubleSide
      })
      
      if (mesh) {
        mesh.material.dispose()
        mesh.material = freshMaterial
        material = freshMaterial
      }
      
      console.log('Material recreated with fresh uniforms')
    } catch (error) {
      console.error('Failed to recreate material:', error)
    }
    
    renderer.render(scene, camera)
    console.log('Forced render complete')
  } else {
    console.error('Renderer not available')
  }
}

const debugShader = () => {
  console.log('Debugging shader...')
  console.log('Material:', material)
  console.log('Uniforms:', uniforms)
  console.log('Traits:', props.traits)
  
  if (material && material.program) {
    const gl = renderer.getContext()
    const program = material.program
    console.log('Shader program:', program)
    
    // Check for shader errors
    if (program.vertexShader) console.log('Vertex shader compiled')
    if (program.fragmentShader) console.log('Fragment shader compiled')
  }
}

const testBasicShader = () => {
  console.log('Testing basic shader without uniforms...')
  
  // Create the simplest possible shader
  const basicVertexShader = `
    varying vec2 vUv;
    
    void main() {
      vUv = uv;
      gl_Position = vec4(position, 1.0);
    }
  `
  
  const basicFragmentShader = `
    precision highp float;
    varying vec2 vUv;
    
    void main() {
      vec2 uv = vUv - 0.5;
      float r = length(uv);
      
      if (r > 0.4) {
        discard;
      }
      
      // Create a rainbow based on angle
      float angle = atan(uv.y, uv.x);
      float normalizedAngle = (angle + 3.14159) / (2.0 * 3.14159);
      
      vec3 color = vec3(
        0.5 + 0.5 * sin(normalizedAngle * 6.28),
        0.5 + 0.5 * sin(normalizedAngle * 6.28 + 2.09),
        0.5 + 0.5 * sin(normalizedAngle * 6.28 + 4.18)
      );
      
      gl_FragColor = vec4(color, 1.0);
    }
  `
  
  try {
    const basicMaterial = new THREE.ShaderMaterial({
      vertexShader: basicVertexShader,
      fragmentShader: basicFragmentShader,
      transparent: true
    })
    
    if (mesh) {
      mesh.material.dispose()
      mesh.material = basicMaterial
      material = basicMaterial
    }
    
    renderer.render(scene, camera)
    console.log('Basic shader test complete - should show rainbow circle')
  } catch (error) {
    console.error('Basic shader test failed:', error)
  }
}

// Setup intersection observer for performance
let observer = null

const setupIntersectionObserver = () => {
  observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      isVisible = entry.isIntersecting
      if (!isVisible && animationId) {
        cancelAnimationFrame(animationId)
        animationId = null
      } else if (isVisible && props.animate && !animationId && !reducedMotion) {
        animate()
      }
    })
  })
  
  if (canvasRef.value) {
    observer.observe(canvasRef.value)
  }
}

// Lifecycle
onMounted(() => {
  checkReducedMotion()
  
  // Listen for reduced motion changes
  const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
  mediaQuery.addEventListener('change', checkReducedMotion)
  
  // Small delay to ensure DOM is ready
  setTimeout(() => {
    initThreeJS()
    setupIntersectionObserver()
  }, 100)
  
  // Add resize listener for perfect circle maintenance
  window.addEventListener('resize', resizeCanvas, { passive: true })
})

onBeforeUnmount(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  if (observer) {
    observer.disconnect()
  }
  if (renderer) {
    renderer.dispose()
  }
  window.removeEventListener('resize', resizeCanvas)
})

watch(() => props.traits, updateUniforms, { deep: true })
watch(() => props.animate, (newVal) => {
  if (newVal && !animationId) {
    animate()
  } else if (!newVal && animationId) {
    cancelAnimationFrame(animationId)
    animationId = null
  }
})
</script>

<style scoped>
.personality-palette-nebula {
  position: relative;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

.palette-container {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 2rem;
  align-items: center;
}

.nebula-canvas {
  display: block;
  width: 100%;
  height: 100%;
  cursor: crosshair;
}

.trait-legend {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.trait-pill {
  display: flex;
  align-items: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
}

.trait-pill--highlighted {
  transform: translateX(8px);
}

.trait-color-ring {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
}

.trait-pill:hover .trait-color-ring {
  transform: scale(1.1);
  filter: brightness(1.2);
}

.trait-inner-circle {
  width: 36px;
  height: 36px;
  background: var(--bg);
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.trait-label {
  font-size: 0.7rem;
  font-weight: 600;
  color: var(--text);
  line-height: 1;
  margin-bottom: 1px;
}

.trait-percentage {
  font-size: 0.6rem;
  font-weight: 500;
  color: var(--muted);
  line-height: 1;
}

.palette-tooltip {
  position: fixed;
  background: rgba(17, 20, 24, 0.95);
  border: 1px solid;
  border-radius: 6px;
  padding: 0.5rem 0.75rem;
  backdrop-filter: blur(12px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
  z-index: 1000;
  pointer-events: none;
  font-size: 0.8rem;
  line-height: 1.4;
}

.tooltip-trait {
  font-weight: 600;
  color: var(--text);
}

.tooltip-strength {
  font-size: 0.75rem;
  color: var(--muted);
}

@media (max-width: 768px) {
  .palette-container {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
  
  .trait-legend {
    flex-direction: row;
    justify-content: center;
    flex-wrap: wrap;
    gap: 0.75rem;
  }
  
  .trait-color-ring {
    width: 45px;
    height: 45px;
  }
  
  .trait-inner-circle {
    width: 32px;
    height: 32px;
  }
  
  .trait-label {
    font-size: 0.65rem;
  }
  
  .trait-percentage {
    font-size: 0.55rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .trait-pill,
  .trait-color-ring {
    transition: none;
  }
  
  .trait-pill:hover .trait-color-ring {
    transform: none;
  }
  
  .trait-pill--highlighted {
    transform: none;
  }
}
</style>

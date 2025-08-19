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

// Fragment shader - Full nebula implementation
const fragmentShader = `
precision highp float;

varying vec2 vUv;

uniform float uTime;
uniform vec2 uResolution;
uniform vec3 uTraitColors[5];
uniform float uTraitWeights[5];
uniform float uRadius;

// Hash function for noise
float hash(vec2 p) {
  p = fract(p * vec2(123.34, 345.45));
  p += dot(p, p + 34.345);
  return fract(p.x * p.y);
}

// Perlin noise
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

// Fractal Brownian Motion
float fbm(vec2 p) {
  float v = 0.0;
  float a = 0.5;
  for (int i = 0; i < 6; i++) {
    v += a * noise(p);
    p *= 2.02;
    a *= 0.5;
  }
  return v;
}

// Worley noise (cellular)
float worley(vec2 p) {
  vec2 i = floor(p);
  float d = 1.0;
  for (int y = -1; y <= 1; y++) {
    for (int x = -1; x <= 1; x++) {
      vec2 g = vec2(float(x), float(y));
      vec2 o = hash(i + g + 0.7) * vec2(0.7, 0.7);
      vec2 r = g + o - fract(p);
      d = min(d, dot(r, r));
    }
  }
  return sqrt(d);
}

// HSV conversion
vec3 rgb2hsv(vec3 c) {
  vec4 K = vec4(0., -1./3., 2./3., -1.);
  vec4 p = mix(vec4(c.bg, K.wz), vec4(c.gb, K.xy), step(c.b, c.g));
  vec4 q = mix(vec4(p.xyw, c.r), vec4(c.r, p.yzx), step(p.x, c.r));
  float d = q.x - min(q.w, q.y);
  float e = 1.0e-10;
  return vec3(abs(q.z + (q.w - q.y) / (6.0 * d + e)), d / (q.x + e), q.x);
}

vec3 hsv2rgb(vec3 c) {
  vec3 p = abs(fract(c.xxx + vec3(0., 1./3., 2./3.)) * 6. - 3.);
  return c.z * mix(vec3(1.), clamp(p - 1., 0., 1.), c.y);
}

// Saturation boost
vec3 sat(vec3 c, float s) {
  vec3 hsv = rgb2hsv(c);
  hsv.y *= s;
  return hsv2rgb(hsv);
}

// Palette blending with trait weights
vec3 paletteBlend(float t, vec3 cols[5], float w[5]) {
  // Spatial distribution biased by field value
  float k0 = w[0] * smoothstep(0.0, 1.0, 1.0 - t);
  float k1 = w[1] * smoothstep(0.0, 1.0, abs(0.5 - t) * 2.0);
  float k2 = w[2] * smoothstep(0.0, 1.0, t);
  float k3 = w[3] * smoothstep(0.0, 1.0, 1.0 - abs(0.5 - t) * 2.0);
  float k4 = w[4] * smoothstep(0.2, 0.8, t);
  
  float sum = k0 + k1 + k2 + k3 + k4 + 1e-5;
  k0 /= sum; k1 /= sum; k2 /= sum; k3 /= sum; k4 /= sum;
  
  return cols[0] * k0 + cols[1] * k1 + cols[2] * k2 + cols[3] * k3 + cols[4] * k4;
}

void main() {
  // Map to NDC centered coordinates
  vec2 uv = vUv - 0.5;
  float r = length(uv);
  
  // Circle mask with soft feather
  if (r > uRadius) {
    discard;
  }
  
  // Domain-warped coordinates for organic flow
  vec2 p = uv * 2.2;
  float n1 = fbm(p + 0.10 * uTime);
  float n2 = fbm(p * 1.7 + vec2(2.0, -1.3) + 0.07 * uTime);
  vec2 warp = vec2(n1 - 0.5, n2 - 0.5);
  vec2 q = p + warp * 1.2;
  
  // Combine Worley for cellular structure
  float cells = worley(q * 1.2);
  float field = fbm(q) * 0.7 + (1.0 - cells) * 0.6;
  field = clamp(field, 0.0, 1.0);
  
  // Get trait colors and blend them
  vec3 color;
  vec3 testColor = uTraitColors[0];
  if (length(testColor) > 0.1) {
    // Use actual trait colors
    vec3 cols[5];
    for (int i = 0; i < 5; i++) cols[i] = uTraitColors[i];
    color = paletteBlend(field, cols, uTraitWeights);
  } else {
    // Fallback nebula colors
    vec3 cols[5];
    cols[0] = vec3(1.0, 0.3, 0.2);  // Red-orange
    cols[1] = vec3(0.2, 0.8, 1.0);  // Cyan
    cols[2] = vec3(0.8, 0.2, 1.0);  // Magenta  
    cols[3] = vec3(0.2, 1.0, 0.4);  // Green
    cols[4] = vec3(1.0, 0.8, 0.2);  // Yellow
    float weights[5];
    weights[0] = 0.2; weights[1] = 0.2; weights[2] = 0.2; weights[3] = 0.2; weights[4] = 0.2;
    color = paletteBlend(field, cols, weights);
  }
  
  // Boost saturation like the reference
  color = sat(color, 1.12);
  
  // Volumetric vignette
  float vign = smoothstep(uRadius * 0.95, uRadius * 0.65, r);
  color *= mix(1.08, 0.82, vign);
  
  // Dense stipple grain texture
  vec2 fragPx = gl_FragCoord.xy;
  float grain1 = fract(sin(dot(fragPx, vec2(12.9898, 78.233))) * 43758.5453);
  float grain2 = noise(q * 9.0 + 13.7);
  float speck = smoothstep(0.55, 0.95, grain2 + grain1 * 0.35);
  
  // Apply stipple effect for pointillism
  color = mix(color * 0.88, color * 1.06, speck);
  
  // Global grain wobble
  float globalGrain = (hash(fragPx + uTime * 60.0) - 0.5) * 2.0;
  color += globalGrain * 0.015;
  
  // Clamp whites to maintain richness (no pure white)
  float maxC = max(max(color.r, color.g), color.b);
  if (maxC > 0.94) {
    color *= 0.94 / maxC;
  }
  
  // Soft edge
  float alpha = smoothstep(uRadius, uRadius * 0.98, r);
  
  gl_FragColor = vec4(color, alpha);
}
`

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
  renderer.setSize(canvasSize.value, canvasSize.value)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
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

// Animation loop
const animate = () => {
  if (!props.animate) return
  
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

// Lifecycle
onMounted(() => {
  // Small delay to ensure DOM is ready
  setTimeout(() => {
    initThreeJS()
  }, 100)
})

onBeforeUnmount(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  if (renderer) {
    renderer.dispose()
  }
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
  width: 100%;
  height: auto;
  cursor: crosshair;
  border-radius: 50%;
  filter: drop-shadow(0 8px 32px rgba(0, 0, 0, 0.4));
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

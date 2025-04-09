<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { ref, shallowRef, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const props = defineProps({
  backgroundColor: {
    type: String,
    default: '#0B071B'
  }
})

const container = ref(null)
const scene = shallowRef(null)
const camera = shallowRef(null)
const renderer = shallowRef(null)
const controls = shallowRef(null)

const init = () => {
  // Create scene
  scene.value = new THREE.Scene()
  scene.value.background = new THREE.Color(props.backgroundColor)
  
  // Create camera
  camera.value = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.value.position.z = 5
  
  // Create renderer
  renderer.value = new THREE.WebGLRenderer({ antialias: true })
  renderer.value.setSize(window.innerWidth, window.innerHeight)
  container.value.appendChild(renderer.value.domElement)
  
  // Add OrbitControls
  controls.value = new OrbitControls(camera.value, renderer.value.domElement)
  controls.value.enableDamping = true
  controls.value.dampingFactor = 0.05
  
  // Add basic lighting
  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(1, 1, 1)
  scene.value.add(light)
  
  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.value.add(ambientLight)
  
  // Handle window resize
  window.addEventListener('resize', onWindowResize)
}

const onWindowResize = () => {
  if (!camera.value || !renderer.value) return
  
  camera.value.aspect = window.innerWidth / window.innerHeight
  camera.value.updateProjectionMatrix()
  renderer.value.setSize(window.innerWidth, window.innerHeight)
}

const animate = () => {
  requestAnimationFrame(animate)
  
  if (!controls.value || !scene.value || !camera.value || !renderer.value) return
  
  // Update controls
  controls.value.update()
  
  renderer.value.render(scene.value, camera.value)
}

onMounted(() => {
  init()
  animate()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onWindowResize)
  
  // Clean up Three.js resources
  if (renderer.value) {
    renderer.value.dispose()
  }
})

defineExpose({
  scene,
  camera,
  renderer,
  controls
})
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
</style> 
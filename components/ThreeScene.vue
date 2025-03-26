<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const container = ref(null)
let scene, camera, renderer, model, controls

const init = () => {
  // Create scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#0B071B')
  
  // Create camera
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.z = 5
  
  // Create renderer
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  container.value.appendChild(renderer.domElement)
  
  // Add OrbitControls
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  
  // Add lights
  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(1, 1, 1)
  scene.add(light)
  
  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)
  
  // Add internal point light
  const internalLight = new THREE.PointLight(0xffffff, 500, 500)
  internalLight.position.set(0, 0, 0)
  scene.add(internalLight)
  
  // Load the GLB model
  const loader = new GLTFLoader()
  loader.load(
    '/models/cry.glb',
    (gltf) => {
      model = gltf.scene
      scene.add(model)
      
      // Center the model
      const box = new THREE.Box3().setFromObject(model)
      const center = box.getCenter(new THREE.Vector3())
      model.position.sub(center)
      
      // Make the model more transparent to see the internal light
      model.traverse((child) => {
        if (child.isMesh) {
          child.material.transparent = true
          child.material.opacity = 0.6
        }
      })
    },
    undefined,
    (error) => {
      console.error('Error loading model:', error)
    }
  )
  
  // Handle window resize
  window.addEventListener('resize', onWindowResize)
}

const onWindowResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

const animate = () => {
  requestAnimationFrame(animate)
  
  // Update controls
  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  init()
  animate()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onWindowResize)
  // Clean up Three.js resources
  if (model) {
    scene.remove(model)
    model.traverse((child) => {
      if (child.isMesh) {
        child.geometry.dispose()
        child.material.dispose()
      }
    })
  }
  renderer.dispose()
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
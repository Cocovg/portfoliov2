<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  },
  radius: {
    type: Number,
    default: 3
  },
  height: {
    type: Number,
    default: 0
  },
  color: {
    type: Number,
    default: 0x0000aa
  },
  initialRotation: {
    type: Number,
    default: 0
  },
  rotationDirection: {
    type: Number,
    default: 1  // 1 for clockwise, -1 for counter-clockwise
  },
  rotationSpeed: {
    type: Number,
    default: 0.001
  }
})

const path = ref(null)

const createPath = () => {
  if (!props.scene) return null;
  
  const group = new THREE.Group()
  const segments = 64 // Number of points in the circle
  const dotSize = 0.02 // Size of each dot
  
  // Create material for dots
  const dotMaterial = new THREE.MeshPhongMaterial({
    color: props.color,
    transparent: true,
    opacity: 0.8,
    emissive: props.color,
    emissiveIntensity: 0.2,
    shininess: 100,
    specular: 0x00ffff
  })
  
  // Create dots along the circle
  for (let i = 0; i < segments; i++) {
    const angle = (i / segments) * Math.PI * 2
    const x = props.radius * Math.cos(angle)
    const z = props.radius * Math.sin(angle)
    
    // Create dot geometry
    const dotGeometry = new THREE.SphereGeometry(dotSize, 8, 8)
    const dot = new THREE.Mesh(dotGeometry, dotMaterial)
    
    // Position the dot
    dot.position.set(x, props.height, z)
    
    // Add dot to group
    group.add(dot)
  }
  
  // Set initial rotation
  group.rotation.y = props.initialRotation
  
  // Add reflective material to path
  const pathMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    shininess: 100,
    specular: 0xffffff,
    reflectivity: 0.5
  })
  group.material = pathMaterial
  
  // Add to scene
  props.scene.add(group)
  
  return group
}

const updatePath = () => {
  if (path.value) {
    path.value.rotation.y += props.rotationSpeed * props.rotationDirection
  }
}

// If scene changes, recreate the path
watch(() => props.scene, (newScene, oldScene) => {
  if (oldScene && path.value) {
    oldScene.remove(path.value)
    disposePath()
  }
  
  if (newScene) {
    path.value = createPath()
  }
}, { immediate: true })

const disposePath = () => {
  if (path.value) {
    path.value.traverse((child) => {
      if (child.isMesh) {
        child.geometry.dispose()
        child.material.dispose()
      }
    })
    path.value = null
  }
}

onMounted(() => {
  if (props.scene && !path.value) {
    path.value = createPath()
  }
})

onBeforeUnmount(() => {
  if (path.value && props.scene) {
    props.scene.remove(path.value)
    disposePath()
  }
})

defineExpose({
  path,
  updatePath
})
</script> 
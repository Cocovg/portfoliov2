<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  },
  starCount: {
    type: Number,
    default: 5000
  },
  size: {
    type: Number,
    default: 1
  },
  opacity: {
    type: Number,
    default: 0.8
  },
  color: {
    type: Number,
    default: 0xffffff
  }
})

let starField

const createStarField = () => {
  const starGeometry = new THREE.BufferGeometry()
  const positions = new Float32Array(props.starCount * 3)
  
  for (let i = 0; i < props.starCount * 3; i += 3) {
    positions[i] = (Math.random() - 0.5) * 2000 // x
    positions[i + 1] = (Math.random() - 0.5) * 2000 // y
    positions[i + 2] = (Math.random() - 0.5) * 2000 // z
  }
  
  starGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  
  const starMaterial = new THREE.PointsMaterial({
    color: props.color,
    size: props.size,
    sizeAttenuation: true,
    transparent: true,
    opacity: props.opacity
  })
  
  starField = new THREE.Points(starGeometry, starMaterial)
  props.scene.add(starField)
  
  return starField
}

const updateStarField = (time) => {
  if (starField) {
    starField.rotation.y += 0.0001
  }
}

onMounted(() => {
  if (props.scene) {
    createStarField()
  }
})

onBeforeUnmount(() => {
  if (starField && props.scene) {
    props.scene.remove(starField)
    starField.geometry.dispose()
    starField.material.dispose()
  }
})

defineExpose({
  starField,
  updateStarField
})
</script> 
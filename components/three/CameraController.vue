<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { ref, onMounted } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  camera: {
    type: Object,
    required: true
  },
  controls: {
    type: Object,
    required: true
  }
})

const isInPointView = ref(false)

const focusCameraOnPoint = (point) => {
  isInPointView.value = true // Set flag when focusing on a point
  const targetPosition = point.position.clone()
  const radius = 5
  const duration = 1500
  const startTime = Date.now()
  const startPosition = props.camera.position.clone()
  
  // Calculate start and target angles
  const startAngle = Math.atan2(startPosition.z, startPosition.x)
  const targetAngle = Math.atan2(targetPosition.z, targetPosition.x)
  
  // Calculate the shortest rotation direction
  let angleDiff = targetAngle - startAngle
  if (angleDiff > Math.PI) angleDiff -= 2 * Math.PI
  if (angleDiff < -Math.PI) angleDiff += 2 * Math.PI
  
  const animateCamera = () => {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)
    
    // Smoother easing function
    const easeProgress = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2
    
    // Calculate current angle
    const currentAngle = startAngle + angleDiff * easeProgress
    
    // Update camera position along the circular path
    props.camera.position.x = radius * Math.cos(currentAngle)
    props.camera.position.z = radius * Math.sin(currentAngle)
    props.camera.position.y = startPosition.y + (targetPosition.y - startPosition.y) * easeProgress
    
    // Always look at the model (center)
    props.controls.target.set(0, 0, 0)
    props.controls.update()
    
    if (progress < 1) {
      requestAnimationFrame(animateCamera)
    }
  }
  
  animateCamera()
  
  return targetPosition
}

const resetCamera = () => {
  isInPointView.value = false // Reset the flag when resetting camera
  const radius = 5
  const duration = 1500
  const startTime = Date.now()
  const startPosition = props.camera.position.clone()
  
  // Calculate start and target angles
  const startAngle = Math.atan2(startPosition.z, startPosition.x)
  const targetAngle = 0 // Reset to front view
  
  // Calculate the shortest rotation direction
  let angleDiff = targetAngle - startAngle
  if (angleDiff > Math.PI) angleDiff -= 2 * Math.PI
  if (angleDiff < -Math.PI) angleDiff += 2 * Math.PI
  
  const animateCamera = () => {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)
    
    // Smoother easing function
    const easeProgress = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2
    
    // Calculate current angle
    const currentAngle = startAngle + angleDiff * easeProgress
    
    // Update camera position along the circular path
    props.camera.position.x = radius * Math.cos(currentAngle)
    props.camera.position.z = radius * Math.sin(currentAngle)
    props.camera.position.y = startPosition.y * (1 - easeProgress) // Smoothly return to center height
    
    // Always look at the model (center)
    props.controls.target.set(0, 0, 0)
    props.controls.update()
    
    if (progress < 1) {
      requestAnimationFrame(animateCamera)
    }
  }
  
  animateCamera()
}

defineExpose({
  isInPointView,
  focusCameraOnPoint,
  resetCamera
})
</script> 
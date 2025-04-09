<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  },
  camera: {
    type: Object,
    required: true
  },
  cameraController: {
    type: Object,
    required: true
  },
  pointMapping: {
    type: Object,
    default: () => ({})
  }
})

const emit = defineEmits(['pointClick', 'pointScreenPosition', 'closePopup'])

const isPaused = ref(false)
const mouseDownPosition = ref(null)
const hasMoved = ref(false)

// Create raycaster
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()

const onMouseDown = (event) => {
  mouseDownPosition.value = {
    x: event.clientX,
    y: event.clientY
  }
  hasMoved.value = false
}

const onMouseMove = (event) => {
  if (mouseDownPosition.value) {
    const dx = event.clientX - mouseDownPosition.value.x
    const dy = event.clientY - mouseDownPosition.value.y
    // If mouse has moved more than 5 pixels, consider it a drag
    if (Math.sqrt(dx * dx + dy * dy) > 5) {
      hasMoved.value = true
    }
  }
}

const onMouseUp = (event) => {
  if (!hasMoved.value) {
    // Only process click if mouse hasn't moved
    onMouseClick(event)
  }
  mouseDownPosition.value = null
  hasMoved.value = false
}

const onMouseClick = (event) => {
  // Calculate mouse position in normalized device coordinates
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
  
  // Update the picking ray with the camera and mouse position
  raycaster.setFromCamera(mouse, props.camera)
  
  // Calculate objects intersecting the picking ray
  const intersects = raycaster.intersectObjects(props.scene.children)
  
  let clickedPoint = false
  
  // Check if we clicked on a point
  for (const intersect of intersects) {
    if (intersect.object.userData.isPoint) {
      const pointPosition = props.cameraController.focusCameraOnPoint(intersect.object)
      clickedPoint = true
      
      // Calculate screen position of the point
      const vector = intersect.object.position.clone()
      vector.project(props.camera)
      const x = (vector.x * 0.5 + 0.5) * window.innerWidth
      const y = (-vector.y * 0.5 + 0.5) * window.innerHeight
      
      // Emit screen position for popup
      emit('pointScreenPosition', { x, y })
      
      // Emit point information
      const pointIndex = intersect.object.userData.pointIndex
      const pathIndex = intersect.object.userData.pathIndex
      const contentIndex = (pathIndex - 1) * 4 + pointIndex
      
      // Use mapping if available
      const pointIdentifier = props.pointMapping[contentIndex] || `point${contentIndex + 1}`
      
      emit('pointClick', pointIdentifier)
      
      // Set pause state
      isPaused.value = true
      
      break
    }
  }
  
  // Only reset if we're in a point view and clicked outside both the popup and points
  if (!clickedPoint && props.cameraController.isInPointView.value) {
    // Check if click was on the popup or its content
    const popupElement = document.querySelector('.popup')
    const popupContent = document.querySelector('.popup-content')
    const imageModal = document.querySelector('.image-modal')
    
    if (popupElement && !popupElement.contains(event.target) && 
        !imageModal?.contains(event.target)) {
      props.cameraController.resetCamera()
      emit('closePopup')
      
      // Resume animations
      isPaused.value = false
    }
  }
}

onMounted(() => {
  // Add event listeners for mouse events
  window.addEventListener('mousedown', onMouseDown)
  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('mouseup', onMouseUp)
})

onBeforeUnmount(() => {
  // Remove event listeners
  window.removeEventListener('mousedown', onMouseDown)
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('mouseup', onMouseUp)
})

defineExpose({
  isPaused
})
</script> 
<template>
  <div v-if="isVisible" class="popup" :style="popupStyle">
    <div class="popup-content">
      <button class="close-button" @click="$emit('close')">×</button>
      <component :is="currentPoint" />
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  isVisible: {
    type: Boolean,
    required: true
  },
  position: {
    type: Object,
    required: true
  },
  currentPoint: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['close'])

const popupStyle = computed(() => {
  // Get viewport dimensions
  const viewportWidth = window.innerWidth
  const viewportHeight = window.innerHeight
  
  // Calculate initial position
  let x = props.position.x
  let y = props.position.y
  
  // Ensure popup stays within viewport bounds
  // Add padding from edges (20px)
  const padding = 20
  
  // Calculate popup dimensions (approximate)
  const popupWidth = 300 // min-width
  const popupHeight = 400 // approximate max-height
  
  // Adjust horizontal position
  if (x < popupWidth / 2 + padding) {
    x = popupWidth / 2 + padding
  } else if (x > viewportWidth - popupWidth / 2 - padding) {
    x = viewportWidth - popupWidth / 2 - padding
  }
  
  // Adjust vertical position
  if (y < popupHeight / 2 + padding) {
    y = popupHeight / 2 + padding
  } else if (y > viewportHeight - popupHeight / 2 - padding) {
    y = viewportHeight - popupHeight / 2 - padding
  }
  
  return {
    left: `${x}px`,
    top: `${y}px`
  }
})
</script>

<style scoped>
.popup {
  position: fixed;
  transform: translate(-50%, -50%);
  z-index: 1000;
  pointer-events: auto;
}

.popup-content {
  background: rgba(11, 7, 27, 0.95);
  border: 1px solid rgba(0, 255, 255, 0.3);
  border-radius: 8px;
  padding: 20px;
  min-width: 600px;
  max-width: 800px;
  max-height: 40vh;
  overflow-y: auto;
  box-shadow: 0 0 20px rgba(0, 255, 255, 0.2);
  position: relative;
  backdrop-filter: blur(10px);
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  color: #00ffff;
  font-size: 24px;
  cursor: pointer;
  padding: 0 8px;
  line-height: 1;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.close-button:hover {
  opacity: 1;
}

.popup-content::-webkit-scrollbar {
  width: 8px;
}

.popup-content::-webkit-scrollbar-track {
  background: rgba(0, 255, 255, 0.1);
  border-radius: 4px;
}

.popup-content::-webkit-scrollbar-thumb {
  background: rgba(0, 255, 255, 0.3);
  border-radius: 4px;
}

.popup-content::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 255, 255, 0.5);
}
</style> 
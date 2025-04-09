<template>
  <div v-if="isVisible" class="popup" :style="popupStyle" @click="handlePopupClick">
    <div class="popup-content" @click.stop>
      <button class="close-button" @click="$emit('close')">×</button>
      <div v-html="renderedContent" class="markdown-content" @click="handleImageClick"></div>
    </div>
  </div>
  <!-- Image zoom modal -->
  <div v-if="zoomedImage" class="image-modal" @click="closeZoomedImage">
    <img :src="zoomedImage" alt="Zoomed image" @click.stop>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { marked } from 'marked'

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
    type: String,
    required: true
  }
})

const emit = defineEmits(['close'])
const content = ref('')

// Import all markdown files
const markdownModules = import.meta.glob('@/assets/content/*.md', { as: 'raw' })

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
  
  // Calculate popup dimensions
  const popupWidth = 500 // min-width increased from 300
  const popupHeight = 400 // approximate max-height
  
  // Adjust horizontal position
  if (x < popupWidth / 2 + padding) {
    x = popupWidth / 2 + padding
  } else if (x > viewportWidth - popupWidth / 2 - padding) {
    x = viewportWidth - popupWidth / 2 - padding
  }
  
  // Always position the popup in the lower half of the viewport
  const minY = viewportHeight * 0.5 // Increased from 0.4 to 0.5 (50% from top)
  const maxY = viewportHeight - popupHeight / 2 - padding // Maximum Y position
  
  // If the click is in the upper half, position the popup at the minimum Y
  if (y < minY) {
    y = minY
  } else if (y > maxY) {
    y = maxY
  }
  
  // Add additional top padding to account for browser UI
  const topOffset = 100 // Additional padding from top
  y = Math.max(y, topOffset)
  
  return {
    left: `${x}px`,
    top: `${y}px`
  }
})

const renderedContent = computed(() => {
  return marked(content.value)
})

const loadContent = async (pointName) => {
  if (!pointName) {
    content.value = ''
    return
  }

  try {
    const filePath = `/assets/content/${pointName}.md`
    const module = markdownModules[filePath]
    if (module) {
      const rawContent = await module()
      content.value = rawContent
    } else {
      console.error('Markdown file not found:', filePath)
      content.value = '# Error\n\nContent not found.'
    }
  } catch (error) {
    console.error('Error loading markdown content:', error)
    content.value = '# Error\n\nFailed to load content.'
  }
}

// Watch for changes in currentPoint
watch(() => props.currentPoint, (newPoint) => {
  loadContent(newPoint)
})

// Load initial content if currentPoint is set
onMounted(() => {
  if (props.currentPoint) {
    loadContent(props.currentPoint)
  }
})

const zoomedImage = ref(null)

const handlePopupClick = (event) => {
  // Only emit close if clicking outside the popup content
  if (!event.target.closest('.popup-content')) {
    emit('close')
  }
}

const handleImageClick = (event) => {
  if (event.target.tagName === 'IMG') {
    event.stopPropagation()
    zoomedImage.value = event.target.src
  }
}

const closeZoomedImage = (event) => {
  event.stopPropagation()
  zoomedImage.value = null
}
</script>

<style src="../assets/styles/points.css"></style>
<style src="../assets/styles/PointPopup.css"></style> 
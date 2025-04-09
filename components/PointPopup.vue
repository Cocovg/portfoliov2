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

<style>
@import '@/assets/styles/points.css';

.popup {
  position: fixed;
  transform: translate(-50%, -50%);
  z-index: 1000;
  pointer-events: none;
}

.popup-content {
  background: rgba(11, 7, 27, 0.95);
  border: 1px solid rgba(0, 255, 255, 0.3);
  border-radius: 8px;
  padding: 20px;
  min-width: 500px;  /* Increased from 300px */
  max-width: 700px;  /* Increased from 500px */
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 0 20px rgba(0, 255, 255, 0.2);
  position: relative;
  backdrop-filter: blur(10px);
  pointer-events: auto;
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

/* Custom scrollbar styling */
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

.markdown-content {
  color: #fff;
  font-family: 'Inter', sans-serif;
}

.markdown-content h1 {
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 16px;
  color: #00ffff;
  text-shadow: 0 0 10px rgba(0, 255, 255, 0.5);
}

.markdown-content p {
  font-size: 16px;
  line-height: 1.6;
  margin-bottom: 16px;
  color: rgba(255, 255, 255, 0.9);
}

.markdown-content h2 {
  font-size: 20px;
  font-weight: 500;
  margin: 24px 0 16px;
  color: #00ffff;
}

.markdown-content ul {
  margin-bottom: 16px;
  padding-left: 20px;
}

.markdown-content li {
  margin-bottom: 8px;
  color: rgba(255, 255, 255, 0.8);
}

.markdown-content code {
  background: rgba(0, 255, 255, 0.1);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Fira Code', monospace;
  font-size: 14px;
  color: #00ffff;
}

.markdown-content pre {
  background: rgba(0, 255, 255, 0.05);
  padding: 16px;
  border-radius: 8px;
  margin: 16px 0;
  overflow-x: auto;
}

.markdown-content pre code {
  background: none;
  padding: 0;
  font-size: 14px;
  line-height: 1.5;
}

.markdown-content img {
  max-width: 80%;
  height: auto;
  display: block;
  margin: 16px auto;
  border-radius: 4px;
  box-shadow: 0 0 10px rgba(0, 255, 255, 0.2);
  cursor: pointer;
  transition: transform 0.2s;
}

.markdown-content img:hover {
  transform: scale(1.02);
}

.image-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  cursor: pointer;
}

.image-modal img {
  max-width: 90%;
  max-height: 90vh;
  object-fit: contain;
  border-radius: 8px;
  box-shadow: 0 0 30px rgba(0, 255, 255, 0.3);
  cursor: default;
}
</style> 
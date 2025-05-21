<template>
  <div v-if="isVisible" class="popup" :style="popupStyle" @click="handlePopupClick">
    <div class="popup-content" @click.stop>
      <button class="close-button" @click="$emit('close')">×</button>
      <div v-html="renderedContent" class="markdown-content" @click="handleContentClick"></div>
    </div>
  </div>
  <!-- Image zoom modal -->
  <div v-if="zoomedImage" class="image-modal" @click="closeZoomedImage">
    <img :src="zoomedImage" alt="Zoomed image" @click.stop class="full-resolution-image">
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

// Configure marked to open links in new tab
marked.setOptions({
  breaks: true,
  gfm: true,
  headerIds: true,
  renderer: (() => {
    const renderer = new marked.Renderer();
    const linkRenderer = renderer.link;
    renderer.link = (href, title, text) => {
      const html = linkRenderer.call(renderer, href, title, text);
      return html.replace(/^<a /, '<a target="_blank" rel="noopener noreferrer" ');
    };
    return renderer;
  })()
});

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
    const response = await fetch(`/content/${pointName}.md`)
    if (response.ok) {
      content.value = await response.text()
    } else {
      console.error('Markdown file not found:', pointName)
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

const handleContentClick = (event) => {
  // Handle image clicks to zoom
  if (event.target.tagName === 'IMG') {
    event.stopPropagation()
    // Use the original image source for full resolution
    zoomedImage.value = event.target.src
  }
  
  // Handle links to open in new tab
  if (event.target.tagName === 'A') {
    event.stopPropagation()
    // Ensure links open in a new tab
    event.target.setAttribute('target', '_blank')
    event.target.setAttribute('rel', 'noopener noreferrer')
  }
}

const closeZoomedImage = (event) => {
  event.stopPropagation()
  zoomedImage.value = null
}
</script>

<style src="../assets/styles/points.css"></style>
<style src="../assets/styles/PointPopup.css"></style>

<style>
/* Add inline styles for full resolution image display */
.image-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  cursor: zoom-out;
}

.full-resolution-image {
  max-width: 90%;
  max-height: 90%;
  object-fit: contain;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.1);
  cursor: default;
}

/* Style for links */
.markdown-content a {
  color: #00ffff;
  text-decoration: none;
  transition: all 0.2s ease;
  border-bottom: 1px solid rgba(0, 255, 255, 0.3);
  padding-bottom: 2px;
}

.markdown-content a:hover {
  color: #ffffff;
  border-bottom-color: #ffffff;
  text-shadow: 0 0 8px rgba(0, 255, 255, 0.5);
}
</style> 
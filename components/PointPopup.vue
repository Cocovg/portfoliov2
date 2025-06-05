<template>
  <div v-if="isVisible" class="popup" :style="popupStyle" @click="handlePopupClick">
    <div class="popup-content" @click.stop>
      <button class="close-button" @click="$emit('close')">×</button>
      <div v-if="renderError" class="error-message">
        <h2>Error Loading Content</h2>
        <p>{{ renderError }}</p>
        <button @click="retryLoad" class="retry-button">Retry</button>
      </div>
      <div v-else-if="renderedContent" v-html="renderedContent" class="markdown-content" @click="handleContentClick"></div>
      <div v-else class="loading">Loading content...</div>
    </div>
  </div>
  <!-- Image zoom modal -->
  <div v-if="zoomedImage" class="image-modal" @click="closeZoomedImage">
    <img :src="zoomedImage" alt="Zoomed image" @click.stop class="full-resolution-image">
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, nextTick, onBeforeUnmount } from 'vue'
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

const emit = defineEmits(['close', 'navigate-to-point'])
const content = ref('')
const renderError = ref(null)
const popupHeight = ref(400) // Default height, will be updated after content is rendered

// Configure marked with minimal settings
marked.setOptions({
  breaks: true,
  gfm: true,
  headerIds: true
});

// Remove custom renderer and handle links directly in the rendered content
const handleLinkClicks = () => {
  // Find all links in the popup content
  const links = document.querySelectorAll('.popup-content a');
  
  // Process each link
  links.forEach(link => {
    const href = link.getAttribute('href');
    
    // Handle internal links (starting with /)
    if (href && href.startsWith('/')) {
      link.addEventListener('click', (event) => {
        event.preventDefault();
        
        // Extract point ID and optional hash
        const path = href.substring(1);
        const [pointId, hash] = path.split('#');
        
        console.log('Internal link clicked:', pointId, hash);
        
        // Navigate to the point and scroll to the hash if present
        emit('navigate-to-point', { pointId, hash });
      });
    } else {
      // External links open in a new tab
      link.setAttribute('target', '_blank');
      link.setAttribute('rel', 'noopener noreferrer');
    }
  });
};

const popupStyle = computed(() => {
  // Get viewport dimensions
  const viewportWidth = window.innerWidth
  const viewportHeight = window.innerHeight
  
  // Calculate popup dimensions
  const popupWidth = 500 // min-width 
  const estimatedHeight = popupHeight.value // Use the measured height or estimate
  
  // Add padding from edges
  const padding = 20
  
  // Start with clicked position
  let x = props.position.x
  let y = props.position.y
  
  // Ensure horizontal positioning stays within bounds
  if (x - (popupWidth / 2) < padding) {
    // Too close to left edge
    x = popupWidth / 2 + padding
  } else if (x + (popupWidth / 2) > viewportWidth - padding) {
    // Too close to right edge
    x = viewportWidth - popupWidth / 2 - padding
  }
  
  // Calculate if popup would extend below or above viewport
  const wouldExtendBelow = (y + (estimatedHeight / 2) > viewportHeight - padding)
  const wouldExtendAbove = (y - (estimatedHeight / 2) < padding)
  
  let transform = 'translate(-50%, -50%)'
  
  if (wouldExtendBelow) {
    // If it would extend below, position it above the click point
    // Use a fixed distance from bottom instead of percentage-based position
    y = viewportHeight - padding - estimatedHeight
    transform = 'translate(-50%, 0)'
  } 
  
  if (wouldExtendAbove) {
    // If it would extend above, position it below the click point
    y = padding
    transform = 'translate(-50%, 0)'
  }
  
  return {
    left: `${x}px`,
    top: `${y}px`,
    transform,
    maxHeight: `${viewportHeight - (padding * 2)}px`
  }
})

// Update the rendered content computed property
const renderedContent = computed(() => {
  if (!content.value) {
    return '';
  }
  
  try {
    // Use default marked configuration
    return marked(content.value);
  } catch (error) {
    console.error('Error rendering markdown:', error);
    renderError.value = `Error rendering content: ${error.message}`;
    return '';
  }
});

// Update popup height and process links after content is rendered
watch(renderedContent, () => {
  if (renderedContent.value) {
    nextTick(() => {
      updatePopupHeight();
      handleLinkClicks();
      
      // Check if there's a hash in the URL to scroll to
      const hash = window.location.hash.substring(1);
      if (hash) {
        console.log('Looking for hash:', hash);
        setTimeout(() => {
          const element = document.getElementById(hash);
          if (element) {
            console.log('Found element with hash, scrolling to it');
            const popupContent = document.querySelector('.popup-content');
            if (popupContent) {
              popupContent.scrollTop = element.offsetTop - 20;
            }
          }
        }, 100);
      }
    });
  }
});

// Update popup height after content is loaded
const updatePopupHeight = () => {
  nextTick(() => {
    const popupElement = document.querySelector('.popup-content')
    if (popupElement) {
      popupHeight.value = popupElement.offsetHeight
    }
  })
}

const retryLoad = () => {
  renderError.value = null;
  loadContent(props.currentPoint);
}

const loadContent = async (pointName) => {
  if (!pointName) {
    console.log('No point name provided')
    content.value = ''
    renderError.value = null
    return
  }

  console.log('Loading content for:', pointName)
  renderError.value = null
  
  try {
    // Simple URL construction with validation
    const contentPath = `/content/${pointName}.md`
    console.log('Fetching from:', contentPath)
    
    const response = await fetch(contentPath)
    console.log('Response status:', response.status)
    
    if (response.ok) {
      const text = await response.text()
      console.log('Content loaded, length:', text.length)
      content.value = text
    } else {
      console.error('Markdown file not found:', pointName, response.status)
      renderError.value = `File not found: ${pointName}.md (Status: ${response.status})`
      content.value = ''
    }
  } catch (error) {
    console.error('Error loading markdown content:', error)
    renderError.value = `Failed to load content: ${error.message}`
    content.value = ''
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
  
  // Add window resize listener to update popup position
  window.addEventListener('resize', updatePopupHeight)
})

onBeforeUnmount(() => {
  // Clean up resize listener
  window.removeEventListener('resize', updatePopupHeight)
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

/* Fix placement issues for popups at bottom of screen */
.popup {
  max-height: 80vh;
  z-index: 999;
}

.popup-content {
  max-height: 70vh;
}

/* Styles for error message */
.error-message {
  padding: 20px;
  color: #ff5555;
  text-align: center;
}

.error-message h2 {
  margin-top: 0;
  font-size: 1.5em;
}

.retry-button {
  background-color: #00ffff;
  color: #000;
  border: none;
  padding: 8px 16px;
  margin-top: 10px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s ease;
}

.retry-button:hover {
  background-color: #ffffff;
  box-shadow: 0 0 8px rgba(0, 255, 255, 0.5);
}

/* Loading indicator */
.loading {
  text-align: center;
  padding: 20px;
  color: #00ffff;
  font-style: italic;
}
</style> 
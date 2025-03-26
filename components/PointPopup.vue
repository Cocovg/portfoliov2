<template>
  <div v-if="isVisible" class="popup-overlay" @click="closePopup">
    <div 
      class="popup-content" 
      @click.stop
      :style="{
        position: 'fixed',
        left: `${position.x}px`,
        top: `${position.y}px`,
        transform: 'translate(-50%, -100%)',
        marginTop: '-20px'
      }"
    >
      <button class="close-button" @click="closePopup">×</button>
      <div class="popup-body">
        <h3>{{ title }}</h3>
        <p>{{ content }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  isVisible: {
    type: Boolean,
    default: false
  },
  title: {
    type: String,
    default: 'Point Information'
  },
  content: {
    type: String,
    default: 'This is some information about the point.'
  },
  position: {
    type: Object,
    default: () => ({ x: 0, y: 0 })
  }
})

const emit = defineEmits(['close'])

const closePopup = () => {
  emit('close')
}
</script>

<style scoped>
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1000;
}

.popup-content {
  background-color: #1a1a1a;
  padding: 1.5rem;
  border-radius: 8px;
  position: relative;
  max-width: 300px;
  width: 90%;
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.1);
  pointer-events: auto;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.close-button {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  background: none;
  border: none;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0.25rem;
  line-height: 1;
}

.close-button:hover {
  color: #888;
}

.popup-body {
  margin-top: 0.5rem;
}

h3 {
  margin: 0 0 0.5rem 0;
  color: #fff;
  font-size: 1.1rem;
}

p {
  margin: 0;
  line-height: 1.4;
  color: #ccc;
  font-size: 0.9rem;
}
</style> 
<template>
  <div 
    class="starting-screen"
    :class="{ 'fade-out': isFading }"
    @transitionend="onTransitionEnd"
  >
    <div class="content">
      <img src="/img/Logo-w.png" alt="Logo" class="logo" />
      <p>Media portfolio - Coco van Glabbeek</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import '../assets/styles/StartingScreen.css'

const emit = defineEmits(['complete'])
const isFading = ref(false)

onMounted(() => {
  // Start fade after 4 seconds
  setTimeout(() => {
    isFading.value = true
  }, 3000)
})

const onTransitionEnd = () => {
  if (isFading.value) {
    emit('complete')
  }
}
</script> 
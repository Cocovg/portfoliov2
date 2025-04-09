<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  },
  modelPath: {
    type: String,
    required: true
  },
  opacity: {
    type: Number,
    default: 0.6
  }
})

const model = ref(null)

const loadModel = () => {
  const loader = new GLTFLoader()
  loader.load(
    props.modelPath,
    (gltf) => {
      model.value = gltf.scene
      props.scene.add(model.value)
      
      // Center the model
      const box = new THREE.Box3().setFromObject(model.value)
      const center = box.getCenter(new THREE.Vector3())
      model.value.position.sub(center)
      
      // Apply transparency
      model.value.traverse((child) => {
        if (child.isMesh) {
          child.material.transparent = true
          child.material.opacity = props.opacity
        }
      })
    },
    undefined,
    (error) => {
      console.error('Error loading model:', error)
    }
  )
}

onMounted(() => {
  if (props.scene) {
    loadModel()
  }
})

onBeforeUnmount(() => {
  if (model.value && props.scene) {
    props.scene.remove(model.value)
    model.value.traverse((child) => {
      if (child.isMesh) {
        child.geometry.dispose()
        child.material.dispose()
      }
    })
  }
})

defineExpose({
  model
})
</script> 
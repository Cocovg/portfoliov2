<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  }
})

const lights = []

const setupLights = () => {
  // Add internal point light
  const internalLight = new THREE.PointLight(0xffffff, 500, 500)
  internalLight.position.set(0, 0, 0)
  props.scene.add(internalLight)
  lights.push(internalLight)

  // Add reflective lights
  const spotLight1 = new THREE.SpotLight(0x00ffff, 100)
  spotLight1.position.set(5, 5, 5)
  spotLight1.angle = Math.PI / 4
  spotLight1.penumbra = 0.1
  spotLight1.decay = 2
  spotLight1.distance = 200
  props.scene.add(spotLight1)
  lights.push(spotLight1)

  const spotLight2 = new THREE.SpotLight(0xff00ff, 100)
  spotLight2.position.set(-5, -5, 5)
  spotLight2.angle = Math.PI / 4
  spotLight2.penumbra = 0.1
  spotLight2.decay = 2
  spotLight2.distance = 200
  props.scene.add(spotLight2)
  lights.push(spotLight2)
}

onMounted(() => {
  if (props.scene) {
    setupLights()
  }
})

onBeforeUnmount(() => {
  // Clean up lights
  lights.forEach(light => {
    if (props.scene) {
      props.scene.remove(light)
    }
  })
})
</script> 
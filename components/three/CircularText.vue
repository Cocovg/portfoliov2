<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  },
  text: {
    type: String,
    required: true
  },
  radius: {
    type: Number,
    default: 3.1
  },
  height: {
    type: Number,
    default: 0
  },
  rotationOffset: {
    type: Number,
    default: 0
  },
  rotationDirection: {
    type: Number,
    default: 1  // 1 for clockwise, -1 for counter-clockwise
  },
  rotationSpeed: {
    type: Number,
    default: 0.001
  },
  fontPath: {
    type: String,
    default: '/fonts/helvetiker_regular.typeface.json'
  },
  color: {
    type: Number,
    default: 0xffffff
  },
  emissiveColor: {
    type: Number,
    default: 0x00ffff
  }
})

let textGroup
let font

// Text configuration
const textConfig = {
  size: 0.07,
  height: 0.05,
  depth: 0.0002,
  curveSegments: 16,
  bevelEnabled: true,
  bevelThickness: 0.03,
  bevelSize: 0.008,
  bevelSegments: 3  
}

const createCircularText = () => {
  const group = new THREE.Group()
  const textMaterial = new THREE.MeshPhongMaterial({
    color: props.color,
    transparent: true,
    opacity: 1,
    emissive: props.emissiveColor,
    emissiveIntensity: 0.4,
    shininess: 100,
    specular: props.emissiveColor
  })

  // Calculate the angle step based on text length with reduced spacing
  const angleStep = (Math.PI * 3) / props.text.length
  
  // Create individual letters and position them along the circle
  for (let i = 0; i < props.text.length; i++) {
    const geometry = new TextGeometry(props.text[i], {
      font: font,
      ...textConfig
    })
    
    // Center the letter geometry
    geometry.computeBoundingBox()
    const centerOffset = new THREE.Vector3()
    geometry.boundingBox.getCenter(centerOffset)
    geometry.translate(-centerOffset.x, -centerOffset.y, -centerOffset.z)
    
    const letter = new THREE.Mesh(geometry, textMaterial)
    
    // Calculate position on circle with adjusted spacing
    const angle = -i * angleStep + props.rotationOffset
    const x = props.radius * Math.cos(angle)
    const z = props.radius * Math.sin(angle)
    
    // Position and rotate letter
    letter.position.set(x, props.height, z)
    
    // Rotate to align with circle and make text readable from outside
    letter.rotation.x = 0
    letter.rotation.y = angle + Math.PI
    letter.rotation.z = 0
    
    group.add(letter)
  }
  
  props.scene.add(group)
  return group
}

const updateText = () => {
  if (textGroup) {
    textGroup.rotation.y += props.rotationSpeed * props.rotationDirection
  }
}

const loadFont = () => {
  return new Promise((resolve) => {
    const fontLoader = new FontLoader()
    fontLoader.load(props.fontPath, (loadedFont) => {
      font = loadedFont
      resolve()
    })
  })
}

onMounted(async () => {
  if (props.scene) {
    await loadFont()
    textGroup = createCircularText()
  }
})

onBeforeUnmount(() => {
  if (textGroup && props.scene) {
    props.scene.remove(textGroup)
    textGroup.traverse((child) => {
      if (child.isMesh) {
        child.geometry.dispose()
        child.material.dispose()
      }
    })
  }
})

defineExpose({
  textGroup,
  updateText
})
</script> 
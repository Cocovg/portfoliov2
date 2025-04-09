<template>
  <!-- This component doesn't have visual elements in the template -->
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'

const props = defineProps({
  scene: {
    type: Object,
    required: true
  },
  camera: {
    type: Object,
    required: true
  },
  font: {
    type: Object,
    required: true
  },
  pathRadius: {
    type: Number,
    default: 3
  },
  pathHeight: {
    type: Number,
    default: 0
  },
  numPoints: {
    type: Number,
    default: 4
  },
  pathIndex: {
    type: Number,
    default: 1
  },
  pathRotation: {
    type: Number,
    default: 0
  },
  startNumber: {
    type: Number,
    default: 1
  },
  verticalOffset: {
    type: Number,
    default: 0.3
  }
})

const emit = defineEmits(['pointClick'])

const points = ref([])

const createPoint = (radius, height, number, angle, pointIndex) => {
  // Create text geometry for the number
  const geometry = new TextGeometry(number.toString(), {
    font: props.font,
    size: 0.25,
    height: 0.05,
    depth: 0.02,
    curveSegments: 12,
    bevelEnabled: true,
    bevelThickness: 0.02,
    bevelSize: 0.02,
    bevelSegments: 5
  })
  
  // Center the geometry
  geometry.computeBoundingBox()
  const centerOffset = new THREE.Vector3()
  geometry.boundingBox.getCenter(centerOffset)
  geometry.translate(-centerOffset.x, -centerOffset.y, -centerOffset.z)
  
  const material = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    transparent: true,
    opacity: 1,
    emissive: 0xffffff,
    emissiveIntensity: 0.8,
    shininess: 100,
    specular: 0xffffff
  })
  
  const point = new THREE.Mesh(geometry, material)
  point.userData.radius = radius
  point.userData.height = height
  point.userData.isPoint = true
  point.userData.pathIndex = props.pathIndex
  point.userData.pointIndex = pointIndex
  point.userData.initialAngle = angle
  
  return point
}

const createPoints = () => {
  const verticalDirection = props.pathIndex === 1 ? -1 : 1
  const heightWithOffset = props.pathHeight + (verticalDirection * props.verticalOffset)
  const angleStep = (2 * Math.PI) / props.numPoints
  
  for (let i = 0; i < props.numPoints; i++) {
    const angle = i * angleStep + props.pathRotation
    const number = props.startNumber + i
    const point = createPoint(props.pathRadius, heightWithOffset, number, angle, i)
    
    points.value.push(point)
    props.scene.add(point)
  }
}

const updatePointPositions = (time) => {
  points.value.forEach(point => {
    const radius = point.userData.radius
    const height = point.userData.height
    const initialAngle = point.userData.initialAngle
    const speed = 0.05
    
    // Determine direction based on path index
    const direction = props.pathIndex === 1 ? 1 : -1
    
    // Calculate new position based on time and direction
    const angle = initialAngle + time * speed * direction
    point.position.x = radius * Math.cos(angle)
    point.position.z = radius * Math.sin(angle)
    point.position.y = height
  })
}

onMounted(() => {
  if (props.scene && props.font) {
    createPoints()
  }
})

onBeforeUnmount(() => {
  if (props.scene) {
    points.value.forEach(point => {
      props.scene.remove(point)
      point.geometry.dispose()
      point.material.dispose()
    })
    points.value = []
  }
})

defineExpose({
  points,
  updatePointPositions
})
</script> 
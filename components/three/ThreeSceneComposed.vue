<template>
  <div ref="sceneContainer" class="three-container">
    <StartingScreen v-if="showStartingScreen" @complete="onStartingScreenComplete" />
    <PointPopup
      :is-visible="showPopup"
      :position="popupPosition"
      :current-point="currentPoint"
      @close="closePopup"
    />
    <PersistentLogo />
    <div class="scene-container">
      <SceneSetup ref="sceneSetup" />
      <template v-if="sceneReady">
        <StarField ref="starField" :scene="scene" />
        <CircularPath ref="upperPath" :scene="scene" :height="1" :initial-rotation="Math.PI/4" :rotation-direction="1" />
        <CircularPath ref="lowerPath" :scene="scene" :height="-1" :initial-rotation="-Math.PI/4" :rotation-direction="-1" />
        <template v-if="font">
          <CircularText
            ref="upperText"
            :scene="scene"
            :font="font"
            :text="'PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - '"
            :height="1.2"
            :rotation-offset="Math.PI/3"
            :rotation-direction="1"
          />
          <CircularText
            ref="lowerText"
            :scene="scene"
            :font="font"
            :text="'LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - '"
            :height="-1.2"
            :rotation-offset="-Math.PI/3"
            :rotation-direction="-1"
          />
          <InteractivePoints
            ref="upperPathPoints"
            :scene="scene"
            :camera="camera"
            :font="font"
            :path-height="1"
            :path-index="1"
            :path-rotation="Math.PI/4"
          />
          <InteractivePoints
            ref="lowerPathPoints"
            :scene="scene"
            :camera="camera"
            :font="font"
            :path-height="-1"
            :path-index="2"
            :path-rotation="-Math.PI/4"
            :start-number="5"
          />
        </template>
        <ModelLoader :scene="scene" model-path="/models/cry.glb" />
        <LightingSystem :scene="scene" />
        <CameraController ref="cameraController" :camera="camera" :controls="controls" />
        <RaycasterController
          ref="raycasterController"
          :scene="scene"
          :camera="camera"
          :camera-controller="cameraControllerRef"
          :point-mapping="pointComponents"
          @point-click="handlePointClick"
          @point-screen-position="handlePointScreenPosition"
          @close-popup="closePopup"
        />
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick, watch } from 'vue'
import * as THREE from 'three'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'
import SceneSetup from './SceneSetup.vue'
import StarField from './StarField.vue'
import CircularPath from './CircularPath.vue'
import CircularText from './CircularText.vue'
import InteractivePoints from './InteractivePoints.vue'
import ModelLoader from './ModelLoader.vue'
import CameraController from './CameraController.vue'
import RaycasterController from './RaycasterController.vue'
import LightingSystem from './LightingSystem.vue'
import PointPopup from '../PointPopup.vue'
import StartingScreen from '../StartingScreen.vue'
import PersistentLogo from '../PersistentLogo.vue'

// Setup scene container
const sceneContainer = ref(null)
const sceneSetup = ref(null)
const cameraController = ref(null)
const raycasterController = ref(null)
const upperPathPoints = ref(null)
const lowerPathPoints = ref(null)
const starField = ref(null)
const upperPath = ref(null)
const lowerPath = ref(null)
const upperText = ref(null)
const lowerText = ref(null)
const font = ref(null)
const sceneReady = ref(false)
let time = 0
let animationFrameId = null

// Get exposed objects for child components
const scene = computed(() => sceneSetup.value?.scene?.value)
const camera = computed(() => sceneSetup.value?.camera?.value)
const controls = computed(() => sceneSetup.value?.controls?.value)
const cameraControllerRef = computed(() => cameraController.value)

// UI state
const showStartingScreen = ref(true)
const showPopup = ref(false)
const popupPosition = ref({ x: 0, y: 0 })
const currentPoint = ref(null)

// Point content mapping
const pointComponents = {
  0: 'point1',
  1: 'point2',
  2: 'point3',
  3: 'point4',
  4: 'point5',
  5: 'point6',
  6: 'point7',
  7: 'point8'
}

// Handle popup related events
const closePopup = () => {
  showPopup.value = false
}

const handlePointScreenPosition = (position) => {
  popupPosition.value = position
}

const handlePointClick = (pointIdentifier) => {
  currentPoint.value = pointIdentifier
  showPopup.value = true
}

const onStartingScreenComplete = () => {
  showStartingScreen.value = false
}

// Load font for text and points
const loadFont = async () => {
  return new Promise((resolve) => {
    const fontLoader = new FontLoader()
    fontLoader.load('/fonts/helvetiker_regular.typeface.json', (loadedFont) => {
      font.value = loadedFont
      resolve(font.value)
    })
  })
}

// Main animation loop
const animate = () => {
  // Update animation time
  if (!raycasterController.value?.isPaused?.value) {
    time += 0.01
  }
  
  // Update star field
  if (starField.value?.updateStarField) {
    starField.value.updateStarField(time)
  }
  
  // Update paths
  if (upperPath.value?.updatePath) {
    upperPath.value.updatePath()
  }
  
  if (lowerPath.value?.updatePath) {
    lowerPath.value.updatePath()
  }
  
  // Update text
  if (upperText.value?.updateText) {
    upperText.value.updateText()
  }
  
  if (lowerText.value?.updateText) {
    lowerText.value.updateText()
  }
  
  // Update points (only if not paused)
  if (!raycasterController.value?.isPaused?.value) {
    if (upperPathPoints.value?.updatePointPositions) {
      upperPathPoints.value.updatePointPositions(time)
    }
    
    if (lowerPathPoints.value?.updatePointPositions) {
      lowerPathPoints.value.updatePointPositions(time)
    }
  }
  
  animationFrameId = requestAnimationFrame(animate)
}

// Initialize the scene, components, and start the render loop
const initScene = async () => {
  // Wait for scene setup to be ready
  if (!scene.value || !camera.value || !controls.value) return
  
  console.log('Scene initialized:', scene.value)
  
  // Load the font
  await loadFont()
  
  // Mark scene as ready to render child components
  sceneReady.value = true
  
  // Wait for child components to be mounted
  await nextTick()
  
  // Start animation loop
  animate()
}

// Watch for scene setup to be ready
watch([scene, camera, controls], ([newScene, newCamera, newControls]) => {
  if (newScene && newCamera && newControls) {
    console.log('Scene objects ready, initializing...')
    initScene()
  }
}, { immediate: true })

onMounted(() => {
  console.log('ThreeSceneComposed mounted')
})

onBeforeUnmount(() => {
  // Cancel animation frame
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
  }
})
</script>

<style>
.three-container {
  width: 100%;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  overflow: hidden;
}

.scene-container {
  width: 100%;
  height: 100%;
  position: relative;
}
</style> 
<template>
  <div ref="container" class="three-container">
    <StartingScreen v-if="showStartingScreen" @complete="onStartingScreenComplete" />
    <PointPopup
      :is-visible="showPopup"
      :position="popupPosition"
      :current-point="currentPoint"
      @close="closePopup"
    />
    <PersistentLogo />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'
import { Raycaster } from 'three'
import PointPopup from './PointPopup.vue'
import StartingScreen from './StartingScreen.vue'
import PersistentLogo from './PersistentLogo.vue'

const container = ref(null)
let scene, camera, renderer, model, controls, starField, paths, points, circularText
let raycaster = new Raycaster()
let mouse = new THREE.Vector2()
let isInPointView = false
let mouseDownPosition = null
let hasMoved = false
let time = 0 // Add time variable for point movement
let font

// Add starting screen state
const showStartingScreen = ref(true)

// Add popup state
const showPopup = ref(false)
const popupPosition = ref({ x: 0, y: 0 })
const currentPoint = ref(null)

// Add pause state
const isPaused = ref(false)

// Update the pointComponents mapping
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

const closePopup = () => {
  showPopup.value = false
}

const createPath = (radius, height, color) => {
  const group = new THREE.Group()
  const segments = 64 // Number of points in the circle
  const dotSize = 0.02 // Size of each dot
  const dotSpacing = 0.1 // Space between dots
  
  // Create material for dots
  const dotMaterial = new THREE.MeshPhongMaterial({
    color: 0x0000aa, // Dark blue color
    transparent: true,
    opacity: 0.8,
    emissive: 0x0000aa, // Dark blue glow
    emissiveIntensity: 0.2,
    shininess: 100,
    specular: 0x00ffff
  })
  
  // Create dots along the circle
  for (let i = 0; i < segments; i++) {
    const angle = (i / segments) * Math.PI * 2
    const x = radius * Math.cos(angle)
    const z = radius * Math.sin(angle)
    
    // Create dot geometry
    const dotGeometry = new THREE.SphereGeometry(dotSize, 8, 8)
    const dot = new THREE.Mesh(dotGeometry, dotMaterial)
    
    // Position the dot
    dot.position.set(x, height, z)
    
    // Add dot to group
    group.add(dot)
  }
  
  return group
}

const createPoint = (radius, height, color, number) => {
  // Create text geometry for the number
  const geometry = new TextGeometry(number.toString(), {
    font: font,
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
  point.userData.color = color
  return point
}

const updatePointPosition = (point, time) => {
  const radius = point.userData.radius
  const height = point.userData.height
  const initialAngle = point.userData.initialAngle
  const speed = 0.05 // Base speed of point movement
  
  // Determine direction based on path index
  const direction = point.userData.pathIndex === 1 ? 1 : -1
  
  // Calculate new position based on time and direction
  const angle = initialAngle + time * speed * direction
  point.position.x = radius * Math.cos(angle)
  point.position.z = radius * Math.sin(angle)
  point.position.y = height
}

const focusCameraOnPoint = (point) => {
  isInPointView = true // Set flag when focusing on a point
  const targetPosition = point.position.clone()
  const radius = 5
  const duration = 1500
  const startTime = Date.now()
  const startPosition = camera.position.clone()
  
  // Calculate start and target angles
  const startAngle = Math.atan2(startPosition.z, startPosition.x)
  const targetAngle = Math.atan2(targetPosition.z, targetPosition.x)
  
  // Calculate the shortest rotation direction
  let angleDiff = targetAngle - startAngle
  if (angleDiff > Math.PI) angleDiff -= 2 * Math.PI
  if (angleDiff < -Math.PI) angleDiff += 2 * Math.PI
  
  const animateCamera = () => {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)
    
    // Smoother easing function
    const easeProgress = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2
    
    // Calculate current angle
    const currentAngle = startAngle + angleDiff * easeProgress
    
    // Update camera position along the circular path
    camera.position.x = radius * Math.cos(currentAngle)
    camera.position.z = radius * Math.sin(currentAngle)
    camera.position.y = startPosition.y + (targetPosition.y - startPosition.y) * easeProgress
    
    // Always look at the model (center)
    controls.target.set(0, 0, 0)
    controls.update()
    
    if (progress < 1) {
      requestAnimationFrame(animateCamera)
    }
  }
  
  animateCamera()
}

const resetCamera = () => {
  isInPointView = false // Reset the flag when resetting camera
  const radius = 5
  const duration = 1500
  const startTime = Date.now()
  const startPosition = camera.position.clone()
  
  // Calculate start and target angles
  const startAngle = Math.atan2(startPosition.z, startPosition.x)
  const targetAngle = 0 // Reset to front view
  
  // Calculate the shortest rotation direction
  let angleDiff = targetAngle - startAngle
  if (angleDiff > Math.PI) angleDiff -= 2 * Math.PI
  if (angleDiff < -Math.PI) angleDiff += 2 * Math.PI
  
  const animateCamera = () => {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)
    
    // Smoother easing function
    const easeProgress = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2
    
    // Calculate current angle
    const currentAngle = startAngle + angleDiff * easeProgress
    
    // Update camera position along the circular path
    camera.position.x = radius * Math.cos(currentAngle)
    camera.position.z = radius * Math.sin(currentAngle)
    camera.position.y = startPosition.y * (1 - easeProgress) // Smoothly return to center height
    
    // Always look at the model (center)
    controls.target.set(0, 0, 0)
    controls.update()
    
    if (progress < 1) {
      requestAnimationFrame(animateCamera)
    }
  }
  
  animateCamera()
}

const onMouseDown = (event) => {
  mouseDownPosition = {
    x: event.clientX,
    y: event.clientY
  }
  hasMoved = false
}

const onMouseMove = (event) => {
  if (mouseDownPosition) {
    const dx = event.clientX - mouseDownPosition.x
    const dy = event.clientY - mouseDownPosition.y
    // If mouse has moved more than 5 pixels, consider it a drag
    if (Math.sqrt(dx * dx + dy * dy) > 5) {
      hasMoved = true
    }
  }
}

const onMouseUp = (event) => {
  if (!hasMoved) {
    // Only process click if mouse hasn't moved
    onMouseClick(event)
  }
  mouseDownPosition = null
  hasMoved = false
}

const onMouseClick = (event) => {
  // Calculate mouse position in normalized device coordinates
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
  
  // Update the picking ray with the camera and mouse position
  raycaster.setFromCamera(mouse, camera)
  
  // Calculate objects intersecting the picking ray
  const intersects = raycaster.intersectObjects(scene.children)
  
  let clickedPoint = false
  
  // Check if we clicked on a point
  for (const intersect of intersects) {
    if (intersect.object.userData.isPoint) {
      focusCameraOnPoint(intersect.object)
      clickedPoint = true
      
      // Calculate screen position of the point
      const vector = intersect.object.position.clone()
      vector.project(camera)
      const x = (vector.x * 0.5 + 0.5) * window.innerWidth
      const y = (-vector.y * 0.5 + 0.5) * window.innerHeight
      
      // Update popup position
      popupPosition.value = { x, y }
      
      // Set the current point component
      const pointIndex = intersect.object.userData.pointIndex
      const pathIndex = intersect.object.userData.pathIndex
      const contentIndex = (pathIndex - 1) * 4 + pointIndex
      currentPoint.value = pointComponents[contentIndex]
      
      showPopup.value = true
      
      // Only pause point movements, not text rotation
      isPaused.value = true
      
      break
    }
  }
  
  // Only reset if we're in a point view and clicked outside both the popup and points
  if (!clickedPoint && isInPointView) {
    // Check if click was on the popup or its content
    const popupElement = document.querySelector('.popup')
    const popupContent = document.querySelector('.popup-content')
    const imageModal = document.querySelector('.image-modal')
    
    if (popupElement && !popupElement.contains(event.target) && 
        !imageModal?.contains(event.target)) {
      resetCamera()
      closePopup()
      currentPoint.value = null
      // Resume animations
      isPaused.value = false
    }
  }
}

const animate = () => {
  requestAnimationFrame(animate)
  
  // Update controls
  controls.update()
  
  // Always rotate paths and text, regardless of pause state
  if (paths && circularText) {
    paths[0].rotation.y += 0.001 // Clockwise rotation for first path
    paths[1].rotation.y -= 0.001 // Counter-clockwise rotation for second path
    
    circularText[0].rotation.y += 0.001 // Rotate upper text with its path
    circularText[1].rotation.y -= 0.001 // Rotate lower text with its path
  }
  
  // Only update point positions if not paused
  if (!isPaused.value) {
    // Update time
    time += 0.01
    
    // Rotate star field slowly
    if (starField) {
      starField.rotation.y += 0.0001
    }

    // Update point positions
    if (points) {
      points.forEach(point => {
        updatePointPosition(point, time)
      })
    }
  }
  
  renderer.render(scene, camera)
}

const onStartingScreenComplete = () => {
  showStartingScreen.value = false
}

const init = () => {
  // Create scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#0B071B')
  
  // Create paths
  const radius = 3 // Distance from center
  const path1Height = 1 // Upper circle
  const path2Height = -1 // Lower circle
  
  // Create paths with different rotations
  const path1 = createPath(radius, path1Height, 0xffffff)
  const path2 = createPath(radius, path2Height, 0xffffff)
  
  // Store paths for animation
  paths = [path1, path2]
  
  // Rotate paths to cross each other
  path1.rotation.y = Math.PI / 4 // 45 degrees
  path2.rotation.y = -Math.PI / 4 // -45 degrees
  
  scene.add(path1)
  scene.add(path2)
  
  // Load font and create points and text
  const fontLoader = new FontLoader()
  fontLoader.load('/fonts/helvetiker_regular.typeface.json', (loadedFont) => {
    font = loadedFont // Store the font for point creation
    
    // Create clickable points
    points = []
    const numPoints = 4
    const angleStep = (2 * Math.PI) / numPoints
    
    // Create points for upper circle (path1)
    for (let i = 0; i < numPoints; i++) {
      const angle = i * angleStep + Math.PI / 4 // Add 45 degrees to match path rotation
      const point = createPoint(radius, path1Height - 0.3, 0xffffff, i + 1)
      point.userData.isPoint = true
      point.userData.pathIndex = 1
      point.userData.pointIndex = i
      point.userData.initialAngle = angle
      points.push(point)
      scene.add(point)
    }
    
    // Create points for lower circle (path2)
    for (let i = 0; i < numPoints; i++) {
      const angle = i * angleStep - Math.PI / 4 // Subtract 45 degrees to match path rotation
      const point = createPoint(radius, path2Height + 0.3, 0xffffff, i + 5)
      point.userData.isPoint = true
      point.userData.pathIndex = 2
      point.userData.pointIndex = i
      point.userData.initialAngle = angle
      points.push(point)
      scene.add(point)
    }
    
    // Create upper circle text
    const upperText = createCircularText('PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - ', 3.1, 1.2, font, Math.PI / 3)
    scene.add(upperText)
    
    // Create lower circle text
    const lowerText = createCircularText('LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - ', 3.1, -1.2, font, -Math.PI / 3)
    scene.add(lowerText)
    
    // Store references for animation
    circularText = [upperText, lowerText]
  })
  
  // Add event listeners for mouse events
  window.addEventListener('mousedown', onMouseDown)
  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('mouseup', onMouseUp)
  
  // Create star field
  const starGeometry = new THREE.BufferGeometry()
  const starCount = 5000 // Number of stars
  const positions = new Float32Array(starCount * 3)
  
  for (let i = 0; i < starCount * 3; i += 3) {
    positions[i] = (Math.random() - 0.5) * 2000 // x
    positions[i + 1] = (Math.random() - 0.5) * 2000 // y
    positions[i + 2] = (Math.random() - 0.5) * 2000 // z
  }
  
  starGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  
  const starMaterial = new THREE.PointsMaterial({
    color: 0xffffff,
    size: 1,
    sizeAttenuation: true,
    transparent: true,
    opacity: 0.8
  })
  
  starField = new THREE.Points(starGeometry, starMaterial)
  scene.add(starField)
  
  // Create camera
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.z = 5
  
  // Create renderer
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  container.value.appendChild(renderer.domElement)
  
  // Add OrbitControls
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  
  // Add lights
  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(1, 1, 1)
  scene.add(light)
  
  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)
  
  // Add internal point light
  const internalLight = new THREE.PointLight(0xffffff, 500, 500)
  internalLight.position.set(0, 0, 0)
  scene.add(internalLight)

  // Add reflective lights
  const spotLight1 = new THREE.SpotLight(0x00ffff, 100)
  spotLight1.position.set(5, 5, 5)
  spotLight1.angle = Math.PI / 4
  spotLight1.penumbra = 0.1
  spotLight1.decay = 2
  spotLight1.distance = 200
  scene.add(spotLight1)

  const spotLight2 = new THREE.SpotLight(0xff00ff, 100)
  spotLight2.position.set(-5, -5, 5)
  spotLight2.angle = Math.PI / 4
  spotLight2.penumbra = 0.1
  spotLight2.decay = 2
  spotLight2.distance = 200
  scene.add(spotLight2)

  // Create reflective materials for paths
  const path1Material = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    shininess: 100,
    specular: 0xffffff,
    reflectivity: 0.5
  })
  path1.material = path1Material

  const path2Material = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    shininess: 100,
    specular: 0xffffff,
    reflectivity: 0.5
  })
  path2.material = path2Material
  
  // Load the GLB model
  const loader = new GLTFLoader()
  loader.load(
    '/models/cry.glb',
    (gltf) => {
      model = gltf.scene
      scene.add(model)
      
      // Center the model
      const box = new THREE.Box3().setFromObject(model)
      const center = box.getCenter(new THREE.Vector3())
      model.position.sub(center)
      
      // Make the model more transparent to see the internal light
      model.traverse((child) => {
        if (child.isMesh) {
          child.material.transparent = true
          child.material.opacity = 0.6
        }
      })
    },
    undefined,
    (error) => {
      console.error('Error loading model:', error)
    }
  )
  
  // Handle window resize
  window.addEventListener('resize', onWindowResize)
}

const onWindowResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// Update text configuration for better proportions
const textConfig = {
  size: 0.07, // Slightly smaller for better fit
  height: 0.05, // Keep thickness
  depth: 0.0002, // Keep depth
  curveSegments: 16,
  bevelEnabled: true,
  bevelThickness: 0.03,
  bevelSize: 0.008,
  bevelSegments: 3  
}

const createCircularText = (text, radius, height, font, rotationOffset = 0) => {
  const group = new THREE.Group()
  const textMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    transparent: true,
    opacity: 1,
    emissive: 0x00ffff,
    emissiveIntensity: 0.4,
    shininess: 100,
    specular: 0x00ffff
  })

  // Calculate the angle step based on text length with reduced spacing
  const angleStep = (Math.PI * 3) / text.length // Reduced from 2π to 1.5π to bring letters closer
  
  // Create individual letters and position them along the circle
  for (let i = 0; i < text.length; i++) {
    const geometry = new TextGeometry(text[i], {
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
    const angle = -i * angleStep + rotationOffset // Reverse the angle direction
    const x = radius * Math.cos(angle)
    const z = radius * Math.sin(angle)
    
    // Position and rotate letter
    letter.position.set(x, height, z)
    
    // Rotate to align with circle and make text readable from outside
    letter.rotation.x = 0 // Keep text horizontal
    letter.rotation.y = angle + Math.PI // Add Math.PI to flip text to be readable
    letter.rotation.z = 0 // Reset any z rotation
    
    group.add(letter)
  }
  
  return group
}

onMounted(() => {
  init()
  animate()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('mousedown', onMouseDown)
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('mouseup', onMouseUp)
  // Clean up Three.js resources
  if (model) {
    scene.remove(model)
    model.traverse((child) => {
      if (child.isMesh) {
        child.geometry.dispose()
        child.material.dispose()
      }
    })
  }
  if (starField) {
    scene.remove(starField)
    starField.geometry.dispose()
    starField.material.dispose()
  }
  if (points) {
    points.forEach(point => {
      scene.remove(point)
      point.geometry.dispose()
      point.material.dispose()
    })
  }
  renderer.dispose()
})
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
</style> 
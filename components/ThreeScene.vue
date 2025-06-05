<template>
  <div ref="container" class="three-container">
    <StartingScreen v-if="showStartingScreen" @complete="onStartingScreenComplete" />
    <PointPopup
      :is-visible="showPopup"
      :position="popupPosition"
      :current-point="currentPoint"
      @close="closePopup"
      @navigate-to-point="handlePointNavigation"
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
import PointPopup from './PointPopup.vue'
import StartingScreen from './StartingScreen.vue'
import PersistentLogo from './PersistentLogo.vue'
import '../assets/styles/ThreeScene.css'

// Constants
const PATHS_ROTATION_SPEED = 0.001
const POINT_MOVEMENT_SPEED = 0.05
const STAR_ROTATION_SPEED = 0.0001
const CAMERA_TRANSITION_DURATION = 1500
const DEFAULT_CAMERA_DISTANCE = 5
const PATH_RADIUS = 3
const UPPER_PATH_HEIGHT = 1
const LOWER_PATH_HEIGHT = -1
const POINT_VERTICAL_OFFSET = 0.3
const DOT_SIZE = 0.02
const PATH_SEGMENTS = 64

// Vue refs
const container = ref(null)
const showStartingScreen = ref(true)
const showPopup = ref(false)
const popupPosition = ref({ x: 0, y: 0 })
const currentPoint = ref(null)
const isPaused = ref(false)

// Three.js objects
let scene, camera, renderer, model, controls, starField, paths, points, circularText
let raycaster = new THREE.Raycaster()
let mouse = new THREE.Vector2()
let font
let time = 0
let isInPointView = false
let mouseDownPosition = null
let hasMoved = false

// Component mappings
const pointComponents = {
  0: 'point1',
  1: 'point2',
  2: 'point3',
  3: 'point4',
  4: 'point5',
  5: 'LO1',
  6: 'LO2', 
  7: 'LO3',
  8: 'LO4',
  9: 'LO5',
}

// Text configuration
const textConfig = {
  size: 0.07,
  height: 0.02,
  depth: 0.00002,
  curveSegments: 16,
  bevelEnabled: true,
  bevelThickness: 0.01,
  bevelSize: 0.005,
  bevelSegments: 3  
}

// Popup management
const closePopup = () => {
  showPopup.value = false
}

// Path creation
const createPath = (radius, height, color) => {
  const group = new THREE.Group()
  
  // Create material for dots
  const dotMaterial = new THREE.MeshPhongMaterial({
    color: 0x0000aa,
    transparent: true,
    opacity: 0.8,
    emissive: 0x0000aa,
    emissiveIntensity: 0.2,
    shininess: 100,
    specular: 0x00ffff
  })
  
  // Create dots along the circle
  for (let i = 0; i < PATH_SEGMENTS; i++) {
    const angle = (i / PATH_SEGMENTS) * Math.PI * 2
    const x = radius * Math.cos(angle)
    const z = radius * Math.sin(angle)
    
    const dotGeometry = new THREE.SphereGeometry(DOT_SIZE, 8, 8)
    const dot = new THREE.Mesh(dotGeometry, dotMaterial)
    
    dot.position.set(x, height, z)
    group.add(dot)
  }
  
  return group
}

// Point creation
const createPoint = (radius, height, color, label) => {
  // Create text geometry for the label
  const geometry = new TextGeometry(label.toString(), {
    font: font,
    size: 0.15,
    height: 0.05,
    depth: 0.0002,
    curveSegments: 12,
    bevelEnabled: true,
    bevelThickness: 0.02,
    bevelSize: 0.009,
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
  point.userData.isBillboard = true // Mark this mesh as a billboard (always facing camera)
  return point
}

// Point position updates
const updatePointPosition = (point, time) => {
  const radius = point.userData.radius
  const height = point.userData.height
  const initialAngle = point.userData.initialAngle
  
  // Determine direction based on path index
  const direction = point.userData.pathIndex === 1 ? 1 : -1
  
  // Calculate new position based on time and direction
  const angle = initialAngle + time * POINT_MOVEMENT_SPEED * direction
  point.position.x = radius * Math.cos(angle)
  point.position.z = radius * Math.sin(angle)
  point.position.y = height
}

// Camera controls
const focusCameraOnPoint = (point) => {
  isInPointView = true
  const targetPosition = point.position.clone()
  const radius = DEFAULT_CAMERA_DISTANCE
  const duration = CAMERA_TRANSITION_DURATION
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
  isInPointView = false
  const duration = CAMERA_TRANSITION_DURATION
  const startTime = Date.now()
  const startPosition = camera.position.clone()
  
  // Define the target position (front view)
  const targetPosition = new THREE.Vector3(0, 0, DEFAULT_CAMERA_DISTANCE)
  
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
    
    // Calculate current position by directly interpolating between start and target
    const newX = startPosition.x + (targetPosition.x - startPosition.x) * easeProgress
    const newY = startPosition.y + (targetPosition.y - startPosition.y) * easeProgress
    const newZ = startPosition.z + (targetPosition.z - startPosition.z) * easeProgress
    
    // Update camera position
    camera.position.set(newX, newY, newZ)
    
    // Always look at the model (center)
    controls.target.set(0, 0, 0)
    controls.update()
    
    if (progress < 1) {
      requestAnimationFrame(animateCamera)
    } else {
      // Ensure we're exactly at the target when done
      camera.position.copy(targetPosition)
      controls.update()
    }
  }
  
  animateCamera()
  // Resume animations when resetting
  isPaused.value = false
}

// Mouse event handlers
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
  const intersects = raycaster.intersectObjects(scene.children, true)
  
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
      const contentIndex = (pathIndex - 1) * 5 + pointIndex
      currentPoint.value = pointComponents[contentIndex]
      
      showPopup.value = true
      
      // Only pause point movements, not text rotation
      isPaused.value = true
      
      break
    }
  }
  
  // Reset view if already in point view and clicked outside points and popups
  if (!clickedPoint && isInPointView) {
    // Check if click was on the popup or its content
    const popupElement = document.querySelector('.popup')
    const popupContent = document.querySelector('.popup-content')
    const imageModal = document.querySelector('.image-modal')
    
    // Check if any of the popup elements exist and contain the click target
    const clickedOnPopup = 
      (popupElement && popupElement.contains(event.target)) ||
      (popupContent && popupContent.contains(event.target)) ||
      (imageModal && imageModal.contains(event.target))
    
    if (!clickedOnPopup) {
      console.log('Resetting camera - clicked outside')
      resetCamera()
      closePopup()
      currentPoint.value = null
      isPaused.value = false
    }
  }
}

// Circular text
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
  const angleStep = (Math.PI * 3) / text.length
  
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
    const angle = -i * angleStep + rotationOffset
    const x = radius * Math.cos(angle)
    const z = radius * Math.sin(angle)
    
    // Position and rotate letter
    letter.position.set(x, height, z)
    
    // Rotate to align with circle and make text readable from outside
    letter.rotation.y = angle + Math.PI
    
    // Mark this letter as billboarded
    letter.userData.isBillboard = true
    
    group.add(letter)
  }
  
  return group
}

// Add this method before the animate() function
const handlePointNavigation = ({ pointId, hash }) => {
  console.log('Navigation requested to:', pointId, hash)
  
  // Find the corresponding point index in the pointComponents mapping
  let targetPointIndex = -1
  let targetPathIndex = -1
  
  // Find the matching point ID in pointComponents
  Object.entries(pointComponents).forEach(([index, id]) => {
    if (id === pointId) {
      // Calculate path and point index based on the index
      const idx = parseInt(index)
      if (idx < 5) { // Points 0-4 are on path 1
        targetPathIndex = 1
        targetPointIndex = idx
      } else { // Points 5-9 are on path 2
        targetPathIndex = 2
        targetPointIndex = idx - 5
      }
    }
  })
  
  console.log('Target indices:', targetPathIndex, targetPointIndex)
  
  // If we found a matching point, focus on it
  if (targetPointIndex >= 0 && targetPathIndex >= 0) {
    // Find the corresponding 3D point object
    const targetPoint = points.find(p => 
      p.userData.pathIndex === targetPathIndex && 
      p.userData.pointIndex === targetPointIndex
    )
    
    console.log('Found target point:', targetPoint ? 'yes' : 'no')
    
    if (targetPoint) {
      // Close the current popup
      closePopup()
      
      // Focus camera on the target point
      focusCameraOnPoint(targetPoint)
      
      // Calculate screen position of the point
      const vector = targetPoint.position.clone()
      vector.project(camera)
      const x = (vector.x * 0.5 + 0.5) * window.innerWidth
      const y = (-vector.y * 0.5 + 0.5) * window.innerHeight
      
      // Update popup position
      popupPosition.value = { x, y }
      
      // Set the current point
      currentPoint.value = pointId
      console.log('Setting current point to:', pointId)
      
      // Show the popup
      showPopup.value = true
      
      // Pause animations
      isPaused.value = true
      
      // If there's a hash, we need to scroll to that element after content loads
      if (hash) {
        // Use setTimeout to ensure content is loaded before attempting to scroll
        setTimeout(() => {
          const element = document.getElementById(hash)
          console.log('Scrolling to element:', hash, element ? 'found' : 'not found')
          if (element) {
            const popupContent = document.querySelector('.popup-content')
            if (popupContent) {
              popupContent.scrollTop = element.offsetTop - 20 // Add some padding
            }
          }
        }, 100)
      }
    }
  }
}

// Animation loop
const animate = () => {
  requestAnimationFrame(animate)
  
  // Update controls
  controls.update()
  
  // Always rotate paths and text, regardless of pause state
  if (paths && circularText) {
    paths[0].rotation.y += PATHS_ROTATION_SPEED
    paths[1].rotation.y -= PATHS_ROTATION_SPEED
    
    circularText[0].rotation.y += PATHS_ROTATION_SPEED
    circularText[1].rotation.y -= PATHS_ROTATION_SPEED
  }
  
  // Only update point positions if not paused
  if (!isPaused.value) {
    // Update time
    time += 0.01
    
    // Rotate star field slowly
    if (starField) {
      starField.rotation.y += STAR_ROTATION_SPEED
    }

    // Update point positions
    if (points) {
      points.forEach(point => {
        updatePointPosition(point, time)
      })
    }
  }
  
  // Make all billboarded objects face the camera
  if (points) {
    points.forEach(point => {
      if (point.userData.isBillboard) {
        // Make the text face the camera
        point.lookAt(camera.position)
      }
    })
  }
  
  // Make all circular text letters face the camera
  if (circularText) {
    circularText.forEach(textGroup => {
      textGroup.children.forEach(letter => {
        if (letter.userData.isBillboard) {
          letter.lookAt(camera.position)
        }
      })
    })
  }
  
  renderer.render(scene, camera)
}

// Initialization and lifecycle
const onStartingScreenComplete = () => {
  showStartingScreen.value = false
}

const initPoints = (fontInstance) => {
  // Create clickable points
  points = []
  const numPoints = 5
  const angleStep = (2 * Math.PI) / numPoints
  
  // Create points for upper circle (path1)
  for (let i = 0; i < numPoints; i++) {
    const angle = i * angleStep + Math.PI / 5
    // Use project names instead of numbers
    const pointLabels = ['Branding', 'UX', 'Development', 'Project X', 'Porftolio']
    const point = createPoint(PATH_RADIUS, UPPER_PATH_HEIGHT - POINT_VERTICAL_OFFSET, 0xffffff, pointLabels[i])
    point.userData.isPoint = true
    point.userData.pathIndex = 1
    point.userData.pointIndex = i
    point.userData.initialAngle = angle
    points.push(point)
    scene.add(point)
  }
  
  // Create points for lower circle (path2)
  const lowerNumPoints = 5
  const lowerAngleStep = (2 * Math.PI) / lowerNumPoints
  
  for (let i = 0; i < lowerNumPoints; i++) {
    const angle = i * lowerAngleStep - Math.PI / 5
    const pointLabel = 'LO' + (i + 1)
    const point = createPoint(PATH_RADIUS, LOWER_PATH_HEIGHT + POINT_VERTICAL_OFFSET, 0xffffff, pointLabel)
    point.userData.isPoint = true
    point.userData.pathIndex = 2
    point.userData.pointIndex = i
    point.userData.initialAngle = angle
    points.push(point)
    scene.add(point)
  }
}

const initCircleTexts = (fontInstance) => {
  // Create upper circle text
  const upperText = createCircularText(
    'PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - PROJECTEN - ', 
    PATH_RADIUS + 0.1, 
    UPPER_PATH_HEIGHT + 0.2, 
    fontInstance, 
    Math.PI / 3
  )
  scene.add(upperText)
  
  // Create lower circle text
  const lowerText = createCircularText(
    'LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - LEERUITKOMSTEN - ', 
    PATH_RADIUS + 0.1, 
    LOWER_PATH_HEIGHT - 0.2, 
    fontInstance, 
    -Math.PI / 3
  )
  scene.add(lowerText)
  
  // Store references for animation
  circularText = [upperText, lowerText]
}

const initStarField = () => {
  const starGeometry = new THREE.BufferGeometry()
  const starCount = 5000
  const positions = new Float32Array(starCount * 3)
  
  for (let i = 0; i < starCount * 3; i += 3) {
    positions[i] = (Math.random() - 0.5) * 2000
    positions[i + 1] = (Math.random() - 0.5) * 2000
    positions[i + 2] = (Math.random() - 0.5) * 2000
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
}

const initLights = () => {
  // Add main directional light
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(1, 1, 1)
  scene.add(directionalLight)
  
  // Add ambient light for overall illumination
  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)
  
  // Add internal point light
  const internalLight = new THREE.PointLight(0xffffff, 500, 500)
  internalLight.position.set(0, 0, 0)
  scene.add(internalLight)

  // Add colored accent lights
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
}

const loadModel = () => {
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
}

const init = () => {
  // Create scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#0B071B')
  
  // Create paths
  const path1 = createPath(PATH_RADIUS, UPPER_PATH_HEIGHT, 0xffffff)
  const path2 = createPath(PATH_RADIUS, LOWER_PATH_HEIGHT, 0xffffff)
  
  // Rotate paths to cross each other
  path1.rotation.y = Math.PI / 4 // 45 degrees
  path2.rotation.y = -Math.PI / 4 // -45 degrees
  
  // Store paths for animation
  paths = [path1, path2]
  
  scene.add(path1)
  scene.add(path2)
  
  // Add reflective materials for paths
  const pathMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    shininess: 100,
    specular: 0xffffff,
    reflectivity: 0.5
  })
  
  // Load font and create points and text
  const fontLoader = new FontLoader()
  fontLoader.load('/fonts/helvetiker_regular.typeface.json', (loadedFont) => {
    font = loadedFont
    initPoints(font)
    initCircleTexts(font)
  })
  
  // Add event listeners for mouse events
  window.addEventListener('mousedown', onMouseDown)
  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('mouseup', onMouseUp)
  
  // Create star field
  initStarField()
  
  // Create camera
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.z = DEFAULT_CAMERA_DISTANCE
  
  // Create renderer with antialiasing for smoother edges
  renderer = new THREE.WebGLRenderer({ 
    antialias: true,
    powerPreference: 'high-performance' 
  })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)) // Limit pixel ratio for performance
  container.value.appendChild(renderer.domElement)
  
  // Add OrbitControls
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.maxDistance = 20
  controls.minDistance = 3
  
  // Add lights
  initLights()
  
  // Load 3D model
  loadModel()
  
  // Handle window resize
  window.addEventListener('resize', onWindowResize)
}

const onWindowResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// Cleanup resources
const cleanupResources = () => {
  // Dispose of geometries and materials
  if (model) {
    scene.remove(model)
    model.traverse((child) => {
      if (child.isMesh) {
        if (child.geometry) child.geometry.dispose()
        if (child.material) {
          if (Array.isArray(child.material)) {
            child.material.forEach(material => material && material.dispose())
          } else {
            child.material.dispose()
          }
        }
      }
    })
  }
  
  if (starField) {
    scene.remove(starField)
    if (starField.geometry) starField.geometry.dispose()
    if (starField.material) starField.material.dispose()
  }
  
  if (points) {
    points.forEach(point => {
      scene.remove(point)
      if (point.geometry) point.geometry.dispose()
      if (point.material) point.material.dispose()
    })
  }
  
  if (circularText) {
    circularText.forEach(textGroup => {
      scene.remove(textGroup)
      textGroup.traverse(child => {
        if (child.isMesh) {
          if (child.geometry) child.geometry.dispose()
          if (child.material) {
            if (Array.isArray(child.material)) {
              child.material.forEach(material => material && material.dispose())
            } else {
              child.material.dispose()
            }
          }
        }
      })
    })
  }
  
  // Dispose of renderer
  if (renderer) renderer.dispose()
}

// Lifecycle hooks
onMounted(() => {
  init()
  animate()
})

onBeforeUnmount(() => {
  // Remove event listeners
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('mousedown', onMouseDown)
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('mouseup', onMouseUp)
  
  // Clean up Three.js resources
  cleanupResources()
})
</script>

<style src="../assets/styles/ThreeScene.css"></style>
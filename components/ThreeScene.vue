<template>
  <div ref="container" class="three-container">
    <PointPopup
      :is-visible="showPopup"
      :title="popupTitle"
      :content="popupContent"
      :position="popupPosition"
      @close="closePopup"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { Raycaster } from 'three'
import PointPopup from './PointPopup.vue'

const container = ref(null)
let scene, camera, renderer, model, controls, starField, paths, points
let raycaster = new Raycaster()
let mouse = new THREE.Vector2()
let isInPointView = false
let mouseDownPosition = null
let hasMoved = false
let time = 0 // Add time variable for point movement

// Add popup state
const showPopup = ref(false)
const popupTitle = ref('')
const popupContent = ref('')

// Add popup position state
const popupPosition = ref({ x: 0, y: 0 })

// Add popup content for each point
const pointContents = [
  {
    title: 'Point 1',
    content: 'This is the first point in the upper circle. It represents the beginning of your journey.'
  },
  {
    title: 'Point 2',
    content: 'The second point symbolizes growth and development in your path.'
  },
  {
    title: 'Point 3',
    content: 'This point represents challenges and obstacles you may encounter.'
  },
  {
    title: 'Point 4',
    content: 'The fourth point shows the culmination of your journey.'
  },
  {
    title: 'Point 5',
    content: 'This point in the lower circle represents reflection and learning.'
  },
  {
    title: 'Point 6',
    content: 'The sixth point symbolizes adaptation and change in your path.'
  },
  {
    title: 'Point 7',
    content: 'This point represents the balance between different aspects of your journey.'
  },
  {
    title: 'Point 8',
    content: 'The final point shows the completion of one cycle and the beginning of another.'
  }
]

const closePopup = () => {
  showPopup.value = false
}

const createPath = (radius, height, color) => {
  const points = []
  const segments = 64 // More segments for smoother circle
  for (let i = 0; i <= segments; i++) {
    const angle = (i / segments) * Math.PI * 2
    points.push(new THREE.Vector3(
      radius * Math.cos(angle),
      height,
      radius * Math.sin(angle)
    ))
  }
  const curve = new THREE.CatmullRomCurve3(points)
  const geometry = new THREE.TubeGeometry(curve, 100, 0.01, 8, false)
  const material = new THREE.MeshPhongMaterial({ color: color })
  return new THREE.Mesh(geometry, material)
}

const createPoint = (radius, angle, height, color) => {
  const geometry = new THREE.SphereGeometry(0.08, 32, 32)
  const material = new THREE.MeshPhongMaterial({ color: color, opacity: 0.2 })
  const point = new THREE.Mesh(geometry, material)
  point.userData.radius = radius // Store radius for animation
  point.userData.height = height // Store height for animation
  point.userData.color = color // Store color for animation
  point.userData.initialAngle = angle // Store initial angle
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
      
      // Show popup with point content
      const pointIndex = intersect.object.userData.pointIndex
      const pathIndex = intersect.object.userData.pathIndex
      const contentIndex = (pathIndex - 1) * 4 + pointIndex
      popupTitle.value = pointContents[contentIndex].title
      popupContent.value = pointContents[contentIndex].content
      showPopup.value = true
      
      break
    }
  }
  
  // Only reset if we're in a point view and didn't click a point
  if (!clickedPoint && isInPointView) {
    resetCamera()
    closePopup()
  }
}

const animate = () => {
  requestAnimationFrame(animate)
  
  // Update controls
  controls.update()
  
  // Update time
  time += 0.01
  
  // Rotate star field slowly
  if (starField) {
    starField.rotation.y += 0.0001
  }

  // Rotate paths in opposite directions
  if (paths) {
    paths[0].rotation.y += 0.001 // Clockwise rotation for first path
    paths[1].rotation.y -= 0.001 // Counter-clockwise rotation for second path
  }

  // Update point positions
  if (points) {
    points.forEach(point => {
      updatePointPosition(point, time)
    })
  }
  
  renderer.render(scene, camera)
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
  
  // Create clickable points
  points = []
  const numPoints = 4
  const angleStep = (2 * Math.PI) / numPoints
  
  // Create points for upper circle (path1)
  for (let i = 0; i < numPoints; i++) {
    const angle = i * angleStep + Math.PI / 4 // Add 45 degrees to match path rotation
    const point = createPoint(radius, angle, path1Height, 0xffffff)
    point.userData.isPoint = true
    point.userData.pathIndex = 1
    point.userData.pointIndex = i
    points.push(point)
    scene.add(point)
  }
  
  // Create points for lower circle (path2)
  for (let i = 0; i < numPoints; i++) {
    const angle = i * angleStep - Math.PI / 4 // Subtract 45 degrees to match path rotation
    const point = createPoint(radius, angle, path2Height, 0xffffff)
    point.userData.isPoint = true
    point.userData.pathIndex = 2
    point.userData.pointIndex = i
    points.push(point)
    scene.add(point)
  }
  
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
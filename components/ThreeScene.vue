<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { Raycaster } from 'three'

const container = ref(null)
let scene, camera, renderer, model, controls, starField, paths, points
let raycaster = new Raycaster()
let mouse = new THREE.Vector2()

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
  const geometry = new THREE.SphereGeometry(0.08, 16, 16)
  const material = new THREE.MeshPhongMaterial({ color: color })
  const point = new THREE.Mesh(geometry, material)
  point.position.x = radius * Math.cos(angle)
  point.position.y = height
  point.position.z = radius * Math.sin(angle)
  return point
}

const focusCameraOnPoint = (point) => {
  const targetPosition = point.position.clone()
  const radius = 5 // Increased from 3 to 5 for better view
  const duration = 1500 // Increased duration for smoother transition
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
  const radius = 5
  const duration = 1500 // Increased duration for smoother transition
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
      break
    }
  }
  
  // If we didn't click a point, reset the camera
  if (!clickedPoint) {
    resetCamera()
  }
}

const init = () => {
  // Create scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#0B071B')
  
  // Create paths
  const radius = 4 // Distance from center
  const path1Height = 1.5 // Upper circle
  const path2Height = -1.5 // Lower circle
  
  // Create paths with different rotations
  const path1 = createPath(radius, path1Height, 0xFFF600)
  const path2 = createPath(radius, path2Height, 0xffffff)
  
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
    const point = createPoint(radius, angle, path1Height, 0xFFF600)
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
  
  // Add event listener for mouse clicks
  window.addEventListener('click', onMouseClick)
  
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

const animate = () => {
  requestAnimationFrame(animate)
  
  // Update controls
  controls.update()
  
  // Rotate star field slowly
  if (starField) {
    starField.rotation.y += 0.0001
  }
  
  renderer.render(scene, camera)
}

onMounted(() => {
  init()
  animate()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('click', onMouseClick)
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
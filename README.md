# 3D Portfolio

A 3D interactive portfolio built with Vue.js and Three.js, featuring an interactive crystal model surrounded by rotating paths with content points.

## Features

- Interactive 3D environment with Three.js
- Rotating circular paths with numbered interaction points
- Content popup system with Markdown support
- Smooth camera animations
- Responsive design

## Structure

### Main Components

- `ThreeScene.vue` - The main 3D scene component that handles all Three.js rendering and interactions
- `PointPopup.vue` - Displays content popups when interaction points are clicked
- `StartingScreen.vue` - Displays the initial loading/intro screen
- `PersistentLogo.vue` - Shows the persistent logo in the corner

### Modular Components (Experimental)

The `components/three/` directory contains an alternative modular implementation of the 3D scene, breaking it down into smaller, specialized components:

- `SceneSetup.vue` - Basic Three.js scene, camera, and renderer setup
- `StarField.vue` - Creates the background star field
- `CircularPath.vue` - Creates the rotating circular paths
- `CircularText.vue` - Creates rotating text around the paths
- `InteractivePoints.vue` - Creates and manages the interactive number points
- `ModelLoader.vue` - Loads the central 3D model
- `CameraController.vue` - Handles camera movements and animations
- `RaycasterController.vue` - Manages mouse interactions with the 3D objects
- `LightingSystem.vue` - Sets up scene lighting
- `ThreeSceneComposed.vue` - Composes all the above components together

## Styling

CSS styles are organized into separate files in `assets/styles/`:

- `global.css` - Basic global styles (body, etc.)
- `ThreeScene.css` - Styles for the 3D scene container
- `PointPopup.css` - Styles for the content popup
- `points.css` - Styles for point content
- `PersistentLogo.css` - Styles for the persistent logo
- `StartingScreen.css` - Styles for the intro screen

## Content

Portfolio content is stored as Markdown files in `assets/content/` with the following naming convention:

- `point1.md` through `point8.md` - Content for each interaction point

## Project Setup

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

## Technologies Used

- Vue.js
- Three.js
- Nuxt.js
- Marked (for Markdown parsing)

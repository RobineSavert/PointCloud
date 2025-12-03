<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import spadeUrl from '../assets/spade.png'

const container = ref<HTMLElement | null>(null)

let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let points: THREE.Points | null = null
let animationId = 0
let controls: OrbitControls | null = null




async function generatePointCloudFromImage(
    img: HTMLImageElement,
    density: number
): Promise<void> {
  if (!scene || !camera) return


  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')!
  canvas.width = img.width
  canvas.height = img.height
  ctx.drawImage(img, 0, 0)

  const { data } = ctx.getImageData(0, 0, img.width, img.height)

  const positions: number[] = []
  const colors: number[] = []

  const step: number = Math.max(1, Math.floor(density))


  for (let y = 0; y < img.height; y += step) {
    for (let x = 0; x < img.width; x += step) {
      let rSum = 0
      let gSum = 0
      let bSum = 0
      let aSum = 0
      let count = 0

      for (let oy = 0; oy < 2; oy++) {
        for (let ox = 0; ox < 2; ox++) {
          const sx = x + ox
          const sy = y + oy
          if (sx >= img.width || sy >= img.height) continue

          const sIdx = (sy * img.width + sx) * 4
          rSum += data[sIdx]
          gSum += data[sIdx + 1]
          bSum += data[sIdx + 2]
          aSum += data[sIdx + 3]
          count++
        }
      }

      if (count === 0) continue

      const r = rSum / count
      const g = gSum / count
      const b = bSum / count
      const a = aSum / count

      const brightness = (r + g + b) / 3

      if (brightness > 250 && a > 230) continue

      if (a < 5) continue

      const px = x - img.width / 2
      const py = -(y - img.height / 2)
      const pz = (Math.random() - 0.5) * 60


      positions.push(px, py, pz)
      colors.push(r / 255, g / 255, b / 255)


      if (step > 6) {
        const jx = px + (Math.random() - 0.5) * step
        const jy = py + (Math.random() - 0.5) * step
        positions.push(jx, jy, pz)
        colors.push(r / 255, g / 255, b / 255)
      }
    }
  }


  if (positions.length === 0) {
    positions.push(0, 0, 0)
    colors.push(1, 1, 1)
  }

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute(
      'position',
      new THREE.Float32BufferAttribute(positions, 3)
  )
  geometry.setAttribute(
      'color',
      new THREE.Float32BufferAttribute(colors, 3)
  )

  const material = new THREE.PointsMaterial({
    size: 0.022,
    vertexColors: true,
    transparent: true,
    opacity: 0.95,
    sizeAttenuation: true,
    depthWrite: false,
    map: null,
  })

  material.onBeforeCompile = (shader) => {
    shader.fragmentShader = shader.fragmentShader.replace(
        `#include <clipping_planes_fragment>`,
        `
      #include <clipping_planes_fragment>
      float dist = length(gl_PointCoord - vec2(0.5));
      if (dist > 0.5) discard;
    `
    );
  }


  if (points) {
    scene.remove(points)
    points.geometry.dispose()
    ;(points.material as THREE.Material).dispose()
    points = null
  }

  points = new THREE.Points(geometry, material)

  const scaleFactor = 1.4 / Math.max(img.width, img.height)
  points.scale.set(scaleFactor, scaleFactor, scaleFactor)

  scene.add(points)


  camera.position.set(0, 0, 2.2)
  camera.updateProjectionMatrix()
  camera.lookAt(0, 0, 0)
}

function animate() {
  animationId = requestAnimationFrame(animate)

  if (points) {
    points.rotation.y += 0.0012
    points.rotation.x += 0.0005
  }

  if (controls) {
    controls.update()
  }

  renderer.render(scene, camera)
}

function onResize() {
  if (!container.value) return
  const w = container.value.clientWidth
  const h = container.value.clientHeight || 1
  camera.aspect = w / h
  camera.updateProjectionMatrix()
  renderer.setSize(w, h)
}

onMounted(() => {
  const el = container.value
  if (!el) return

  scene = new THREE.Scene()

  const w = el.clientWidth || window.innerWidth
  const h = el.clientHeight || window.innerHeight

  camera = new THREE.PerspectiveCamera(45, w / h, 0.1, 100)
  camera.position.z = 2.2

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setSize(w, h)
  el.appendChild(renderer.domElement)

  // ⭐ ORBIT CONTROLS SETUP (THIS WAS MISSING)
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.rotateSpeed = 0.4
  controls.enablePan = false
  controls.minDistance = 0.4
  controls.maxDistance = 6

  window.addEventListener('resize', onResize)

  // ⭐ DEFAULT SPADE RENDERING
  const defaultImg = new Image()
  defaultImg.src = spadeUrl
  defaultImg.onload = () => {
    console.log("Default spade loaded")
    generatePointCloudFromImage(defaultImg, 6)
  }

  animate()
})


onBeforeUnmount(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('resize', onResize)
  renderer.dispose()
  if (controls) {
    controls.dispose()
  }
})

defineExpose({
  generatePointCloudFromImage,
})
</script>

<template>
  <div ref="container" class="w-full h-full bg-neutral-950"></div>
</template>

<style scoped>
div {
  overflow: hidden;
}
</style>

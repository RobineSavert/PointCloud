<script setup lang="ts">
  import { onBeforeUnmount, onMounted, ref } from 'vue';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
  import spadeUrl from '../assets/spade.png';

  const container = ref<HTMLElement | null>(null);

  let scene: THREE.Scene;
  let camera: THREE.PerspectiveCamera;
  let renderer: THREE.WebGLRenderer;
  let points: THREE.Points | null = null;
  let controls: OrbitControls | null = null;

  let animationId = 0;

  const emit = defineEmits<{
    (e: 'defaultLoaded', img: HTMLImageElement): void;
  }>();

  async function generatePointCloudFromImage(
    img: HTMLImageElement,
    density: number,
    pointSize: number,
    colorMode: string = 'original',
    tintColor: string = '#ffffff',
    opacity: number = 1,
    depthScale: number = 0,
    depthMode: 'noise' | 'brightness' = 'noise'
  ): Promise<void> {
    if (!scene || !camera) return;

    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d')!;
    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);

    const { data } = ctx.getImageData(0, 0, img.width, img.height);

    const positions: number[] = [];
    const colors: number[] = [];
    const step = Math.max(1, Math.floor(density));

    for (let y = 0; y < img.height; y += step) {
      for (let x = 0; x < img.width; x += step) {
        let rSum = 0,
          gSum = 0,
          bSum = 0,
          aSum = 0,
          count = 0;

        for (let oy = 0; oy < 2; oy++) {
          for (let ox = 0; ox < 2; ox++) {
            const sx = x + ox;
            const sy = y + oy;
            if (sx >= img.width || sy >= img.height) continue;

            const idx = (sy * img.width + sx) * 4;
            rSum += data[idx]!;
            gSum += data[idx + 1]!;
            bSum += data[idx + 2]!;
            aSum += data[idx + 3]!;
            count++;
          }
        }

        if (!count) continue;

        const a = aSum / count;
        let r = rSum / count;
        let g = gSum / count;
        let b = bSum / count;

        const brightness = (r + g + b) / 3;

        switch (colorMode) {
          case 'mono':
            r = g = b = brightness;
            break;
          case 'invert':
            r = 255 - r;
            g = 255 - g;
            b = 255 - b;
            break;
          case 'tint':
            const tr = parseInt(tintColor.slice(1, 3), 16);
            const tg = parseInt(tintColor.slice(3, 5), 16);
            const tb = parseInt(tintColor.slice(5, 7), 16);
            const f = brightness / 255;
            r = tr * f;
            g = tg * f;
            b = tb * f;
            break;
        }

        if (brightness > 250 && a > 230) continue;
        if (a < 5) continue;

        const px = x - img.width / 2;
        const py = -(y - img.height / 2);

        let pz = 0;

        if (depthMode === 'brightness') {
          const depth = (brightness / 255 - 0.5) * depthScale * 200;
          const jitter = (Math.random() - 0.5) * 5;
          pz = depth + jitter;
        } else {
          pz = (Math.random() - 0.5) * depthScale * 60;
        }

        positions.push(px, py, pz);
        colors.push(r / 255, g / 255, b / 255);

        if (step > 6) {
          const jx = px + (Math.random() - 0.5) * step;
          const jy = py + (Math.random() - 0.5) * step;
          positions.push(jx, jy, pz);
          colors.push(r / 255, g / 255, b / 255);
        }
      }
    }

    if (positions.length === 0) {
      positions.push(0, 0, 0);
      colors.push(1, 1, 1);
    }

    const geometry = new THREE.BufferGeometry();
    geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
    geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));

    const material = new THREE.PointsMaterial({
      size: pointSize,
      vertexColors: true,
      transparent: true,
      opacity: opacity,
      sizeAttenuation: true,
      depthWrite: false,
      map: null,
    });

    material.onBeforeCompile = (shader) => {
      shader.fragmentShader = shader.fragmentShader.replace(
        `#include <clipping_planes_fragment>`,
        `
      #include <clipping_planes_fragment>
      float dist = length(gl_PointCoord - vec2(0.5));
      if (dist > 0.5) discard;
    `
      );
    };

    if (points) {
      scene.remove(points);
      points.geometry.dispose();
      (points.material as THREE.Material).dispose();
    }

    points = new THREE.Points(geometry, material);

    const scaleFactor = 1.4 / Math.max(img.width, img.height);
    points.scale.set(scaleFactor, scaleFactor, scaleFactor);
    scene.add(points);

    camera.position.set(0, 0, 2.2);
    camera.lookAt(0, 0, 0);
    camera.updateProjectionMatrix();
  }

  function animate() {
    animationId = requestAnimationFrame(animate);

    if (points) {
      points.rotation.y += 0.0012;
      points.rotation.x += 0.0005;
    }

    controls?.update();
    renderer.render(scene, camera);
  }

  function onResize() {
    if (!container.value) return;
    const w = container.value.clientWidth;
    const h = container.value.clientHeight || 1;
    camera.aspect = w / h;
    camera.updateProjectionMatrix();
    renderer.setSize(w, h);
  }

  onMounted(() => {
    const el = container.value;
    if (!el) return;

    scene = new THREE.Scene();

    const w = el.clientWidth || window.innerWidth;
    const h = el.clientHeight || window.innerHeight;

    camera = new THREE.PerspectiveCamera(45, w / h, 0.1, 100);
    camera.position.z = 2.2;

    renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(w, h);
    el.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.rotateSpeed = 0.4;
    controls.enablePan = false;
    controls.minDistance = 0.4;
    controls.maxDistance = 6;

    window.addEventListener('resize', onResize);

    const img = new Image();
    img.src = spadeUrl;
    img.onload = () => {
      generatePointCloudFromImage(img, 6, 0.008, 'original', '#ffffff', 1, 0, 'noise');
      emit('defaultLoaded', img);
    };

    animate();
  });

  onBeforeUnmount(() => {
    cancelAnimationFrame(animationId);
    window.removeEventListener('resize', onResize);
    renderer.dispose();
    controls?.dispose();
  });

  defineExpose({
    generatePointCloudFromImage,
  });
</script>

<template>
  <div ref="container" class="h-full w-full bg-neutral-950"></div>
</template>

<style scoped>
  div {
    overflow: hidden;
  }
</style>

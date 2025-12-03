<script setup lang="ts">
import ImageUploader from './components/ImageUploader.vue'
import PointCloudCanvas from './components/PointCloudCanvas.vue'
import { ref, watch } from 'vue'

const lastImage = ref<HTMLImageElement | null>(null)




type CloudAPI = {
  generatePointCloudFromImage: (img: HTMLImageElement, density: number) => void
}


const density = ref<number>(6)
const cloud = ref<CloudAPI | null>(null)

function onImageLoaded(img: HTMLImageElement) {
  lastImage.value = img
  if (cloud.value) {
    cloud.value.generatePointCloudFromImage(img, density.value)
  }
}


watch(density, (newVal) => {
  if (lastImage.value && cloud.value) {
    cloud.value.generatePointCloudFromImage(lastImage.value, newVal)
  }
})


</script>

<template>

  <div class="w-screen h-screen bg-neutral-900 text-white overflow-hidden flex flex-col md:flex-row">


    <div
        class="w-full md:w-64 p-6 border-b md:border-b-0 md:border-r border-white/20
         flex flex-col gap-4"
    >
      <h1 class="text-xl font-semibold">Uploader</h1>
      <p class="text-sm text-white/70">Choose an image to turn it into a point cloud.</p>

      <ImageUploader @imageLoaded="onImageLoaded" />

      <label class="text-sm mt-4 text-white/70">Density: {{ density }}</label>
      <input
          type="range"
          min="3"
          max="20"
          step="1"
          v-model="density"
          class="w-full"
      />
    </div>


    <div class="flex-1 min-h-0">
      <PointCloudCanvas ref="cloud" />
    </div>

  </div>
</template>

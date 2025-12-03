<script setup lang="ts">
  import ImageUploader from './components/ImageUploader.vue';
  import PointCloudCanvas from './components/PointCloudCanvas.vue';
  import { ref, watch } from 'vue';

  const lastImage = ref<HTMLImageElement | null>(null);

  type CloudAPI = {
    generatePointCloudFromImage: (img: HTMLImageElement, density: number) => void;
  };

  const density = ref<number>(6);
  const cloud = ref<CloudAPI | null>(null);

  function onImageLoaded(img: HTMLImageElement) {
    lastImage.value = img;
    if (cloud.value) {
      cloud.value.generatePointCloudFromImage(img, density.value);
    }
  }

  watch(density, (newVal) => {
    if (lastImage.value && cloud.value) {
      cloud.value.generatePointCloudFromImage(lastImage.value, newVal);
    }
  });
</script>

<template>
  <div
    class="flex h-screen w-screen flex-col overflow-hidden bg-neutral-900 text-white md:flex-row"
  >
    <div
      class="flex w-full flex-col gap-4 border-b border-white/20 p-6 md:w-64 md:border-r md:border-b-0"
    >
      <h1 class="text-xl font-semibold">Uploader</h1>
      <p class="text-sm text-white/70">Choose an image to turn it into a point cloud.</p>

      <ImageUploader @imageLoaded="onImageLoaded" />

      <div class="w-full flex flex-col gap-2 mt-4">
        <label class="text-xs tracking-wide text-white/60 mb-2">Density: {{ density }}</label>

        <input
            type="range"
            min="3"
            max="20"
            step="1"
            v-model="density"
            class="w-full h-1 rounded-lg appearance-none cursor-pointer
           bg-white/10 accent-white"
        />
      </div>
    </div>
      <div class="min-h-0 flex-1">
      <PointCloudCanvas ref="cloud" @defaultLoaded="onImageLoaded" />
    </div>
  </div>
</template>

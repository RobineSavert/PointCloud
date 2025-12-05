<script setup lang="ts">
  import ImageUploader from './components/ImageUploader.vue';
  import PointCloudCanvas from './components/PointCloudCanvas.vue';
  import { ref, watch } from 'vue';

  const lastImage = ref<HTMLImageElement | null>(null);

  type CloudAPI = {
    generatePointCloudFromImage: (
      img: HTMLImageElement,
      density: number,
      pointSize: number,
      colorMode: string,
      tintColor: string,
      opacity: number,
      depthScale: number,
      depthMode: 'noise' | 'brightness',
      scatterEnabled: boolean
    ) => void;
  };

  const density = ref<number>(6);
  const pointSize = ref<number>(0.006);
  const colorMode = ref<'original' | 'mono' | 'tint' | 'invert'>('original');
  const tintColor = ref('#ffffff');
  const opacity = ref<number>(0.95);
  const depthScale = ref<number>(0);
  const depthMode = ref<'noise' | 'brightness'>('noise');
  const scatterEnabled = ref<boolean>(false);

  const cloud = ref<CloudAPI | null>(null);

  function updateCloud() {
    if (lastImage.value && cloud.value) {
      cloud.value.generatePointCloudFromImage(
        lastImage.value,
        density.value,
        pointSize.value,
        colorMode.value,
        tintColor.value,
        opacity.value,
        depthScale.value,
        depthMode.value,
        scatterEnabled.value
      );
    }
  }

  function onImageLoaded(img: HTMLImageElement) {
    lastImage.value = img;
    updateCloud();
  }

  watch(density, updateCloud);
  watch(pointSize, updateCloud);
  watch([colorMode, tintColor], updateCloud);
  watch(opacity, updateCloud);
  watch(depthScale, updateCloud);
  watch(depthMode, updateCloud);
  watch(scatterEnabled, updateCloud);
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

      <div class="mt-4 grid grid-cols-3 gap-4 md:flex md:flex-col md:gap-4">
        <div class="flex flex-col gap-2">
          <label class="mb-2 text-xs tracking-wide text-white/60">Density: {{ density }}</label>
          <input
            type="range"
            min="3"
            max="20"
            step="1"
            v-model="density"
            class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-white/10 accent-white"
          />
        </div>

        <div class="flex flex-col gap-2">
          <label class="mb-2 text-xs tracking-wide text-white/60">
            Point size: {{ pointSize }}
          </label>
          <input
            type="range"
            min="0.008"
            max="0.05"
            step="0.001"
            v-model="pointSize"
            class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-white/10 accent-white"
          />
        </div>

        <div class="flex flex-col gap-2">
          <label class="mb-2 text-xs tracking-wide text-white/60">
            Opacity: {{ opacity.toFixed(2) }}
          </label>
          <input
            type="range"
            min="0.1"
            max="1"
            step="0.01"
            v-model.number="opacity"
            class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-white/10 accent-white"
          />
        </div>

        <div class="flex flex-col gap-2">
          <label class="mb-2 text-xs tracking-wide text-white/60">
            Depth: {{ depthScale.toFixed(2) }}
          </label>
          <input
            type="range"
            min="0"
            max="2"
            step="0.01"
            v-model.number="depthScale"
            class="h-1 w-full cursor-pointer appearance-none rounded-lg bg-white/10 accent-white"
          />
        </div>

        <div class="flex flex-col gap-2">
          <label class="mb-2 text-xs tracking-wide text-white/60">Depth mode</label>
          <select
            v-model="depthMode"
            class="h-8 w-full rounded-lg bg-white/10 px-2 text-sm text-white accent-white"
          >
            <option value="noise">Noise (dreamy)</option>
            <option value="brightness">Brightness depth</option>
          </select>
        </div>

        <div class="flex flex-col gap-2">
          <label class="mb-2 text-xs tracking-wide text-white/60">Color mode</label>
          <select
            v-model="colorMode"
            class="h-8 w-full rounded-lg bg-white/10 px-2 text-sm text-white accent-white"
          >
            <option value="original">Original</option>
            <option value="mono">Monochrome</option>
            <option value="tint">Tint</option>
            <option value="invert">Invert</option>
          </select>
        </div>

        <div v-if="colorMode === 'tint'" class="col-span-3 flex flex-col gap-2 md:col-span-1">
          <label class="mb-2 text-xs tracking-wide text-white/60">Tint color</label>
          <input
            type="color"
            v-model="tintColor"
            class="h-8 w-full cursor-pointer rounded-lg bg-white/10 accent-white"
          />
        </div>
      </div>
    </div>
    <div class="min-h-0 flex-1">
      <PointCloudCanvas ref="cloud" @defaultLoaded="onImageLoaded" />
    </div>
  </div>
</template>

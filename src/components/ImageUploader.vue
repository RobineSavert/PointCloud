<script setup lang="ts">
  const emit = defineEmits<{
    (e: 'imageLoaded', img: HTMLImageElement): void;
  }>();

  function handleFile(e: Event) {
    const file = (e.target as HTMLInputElement).files?.[0];
    if (!file) return;

    const img = new Image();
    img.onload = () => emit('imageLoaded', img);
    img.src = URL.createObjectURL(file);
  }
</script>

<template>
  <label
    class="group relative inline-flex cursor-pointer items-center gap-2 overflow-hidden rounded-lg border border-white/15 bg-gradient-to-br from-white/10 to-white/5 px-6 py-2 font-light tracking-wide text-white/80 transition-all duration-300 hover:text-white"
  >
    <span
      class="absolute inset-0 translate-x-[-100%] bg-gradient-to-r from-transparent via-white/20 to-transparent transition-transform duration-700 ease-out group-hover:translate-x-[100%]"
    ></span>

    <svg
      class="relative z-10 h-4 w-4 opacity-80"
      fill="none"
      stroke="currentColor"
      stroke-width="2"
      viewBox="0 0 24 24"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M4 16v2a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-2m-4-4-4-4m0 0-4 4m4-4v12"
      />
    </svg>

    <span class="relative z-10 text-sm">Upload image</span>

    <input type="file" accept="image/*" class="hidden" @change="handleFile" />
  </label>
</template>

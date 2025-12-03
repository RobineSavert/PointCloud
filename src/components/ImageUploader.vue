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
      class="relative cursor-pointer inline-flex items-center gap-2 px-6 py-2 rounded-lg
         bg-gradient-to-br from-white/10 to-white/5 border border-white/15
         text-white/80 font-light tracking-wide overflow-hidden
         hover:text-white transition-all duration-300 group"
  >
    <!-- shine -->
    <span
        class="absolute inset-0 bg-gradient-to-r from-transparent via-white/20 to-transparent
           translate-x-[-100%] group-hover:translate-x-[100%]
           transition-transform duration-700 ease-out"
    ></span>

    <svg class="w-4 h-4 relative z-10 opacity-80" fill="none" stroke="currentColor" stroke-width="2"
         viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round"
            d="M4 16v2a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-2m-4-4-4-4m0 0-4 4m4-4v12" />
    </svg>

    <span class="relative z-10 text-sm">Upload image</span>

    <input
        type="file"
        accept="image/*"
        class="hidden"
        @change="handleFile"
    />
  </label>


</template>

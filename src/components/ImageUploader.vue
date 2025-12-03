<script setup lang="ts">
const emit = defineEmits<{
  (e: 'imageLoaded', img: HTMLImageElement): void
}>()

function handleFile(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return

  const img = new Image()
  img.onload = () => emit('imageLoaded', img)
  img.src = URL.createObjectURL(file)
}
</script>

<template>
  <input
      type="file"
      accept="image/*"
      @change="handleFile"
      class="text-white cursor-pointer"
  />
</template>

<script setup>
const props = defineProps(["width"]);
</script>

<template>
  <div class="w-full bg-gray-200 h-2.5">
    <div class="bg-blue-600 h-2.5" :style="{ width: `${width}%` }"></div>
  </div>
</template>

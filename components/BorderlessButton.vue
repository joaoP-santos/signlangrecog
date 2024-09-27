<script setup>
const props = defineProps(["text"]);
</script>

<template>
  <div
    class="px-8 py-4 border border-blue text-blue font-bold rounded hover:bg-blue hover:text-bright transition duration-300"
  >
    {{ text }}
  </div>
</template>

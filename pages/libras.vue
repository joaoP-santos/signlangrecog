<script setup>
useHead({
  title: "LIBRAS Alphabet",
  meta: [
    { name: "description", content: "LIBRAS Alphabet" },
    { name: "viewport", content: "width=device-width, initial-scale=1.0" },
  ],
});

setPageLayout("recog");

useSeoMeta({
  title: "LIBRAS Alphabet",
  ogTitle: "LIBRAS Alphabet",
  description:
    "A real-time Brazilian Sign Language (LIBRAS) alphabet recognizer",
  ogDescription:
    "A real-time Brazilian Sign Language (LIBRAS) alphabet recognizer",
});
</script>

<template>
  <NuxtLayout modelAssetPath="/libras-alpha-v2.task" imagesDir="libras"
    >LIBRAS</NuxtLayout
  >
</template>

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

const modelAssetPath = "/libras-alpha-v1.task";
</script>

<template>
  <NuxtLayout modelAssetPath="/libras-alpha-v1.task" imagesDir="libras"
    >LIBRAS</NuxtLayout
  >
</template>

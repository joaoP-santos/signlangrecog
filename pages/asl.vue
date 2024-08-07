<script setup>
useHead({
  title: "ASL Alphabet",
  meta: [
    { name: "description", content: "ASL Alphabet" },
    { name: "viewport", content: "width=device-width, initial-scale=1.0" },
  ],
});

setPageLayout("recog");

useSeoMeta({
  title: "ASL Alphabet",
  ogTitle: "ASL Alphabet",
  description: "A real-time American Sign Language (ASL) alphabet recognizer",
  ogDescription: "A real-time American Sign Language (ASL) alphabet recognizer",
});
</script>

<template>
  <NuxtLayout modelAssetPath="/asl-alpha.task" imagesDir="asl">ASL</NuxtLayout>
</template>

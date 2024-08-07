<script setup>
useHead({
  title: "Sign Language Recognition",
  meta: [
    { name: "description", content: "ASL Alphabet" },
    { name: "viewport", content: "width=device-width, initial-scale=1.0" },
  ],
});

useSeoMeta({
  title: "Sign Language Recognition",
  ogTitle: "Sign Language Recognition",
});
</script>
<template>
  <div
    class="flex flex-col gap-2 justify-center items-center self-center h-svh"
  >
    <h1 class="text-5xl font-light text-center">
      Let's start LIVE sign language recognizing
    </h1>
    <p class="text-center">But first, pick your language</p>
    <div
      class="flex flex-row gap-8 *:border *:p-4 *:w-24 *:text-center *:border-blue-400"
    >
      <NuxtLink to="/libras">LIBRAS</NuxtLink>
      <NuxtLink to="/asl">ASL</NuxtLink>
    </div>
  </div>
</template>

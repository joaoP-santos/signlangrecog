<template>
  <div
    class="flex flex-col gap-2 justify-center items-center self-center h-svh"
  >
    <h1 class="text-5xl font-light">
      Let's start LIVE sign language recognizing
    </h1>
    <p>But first, pick your language</p>
    <div
      class="flex flex-row gap-8 *:border *:p-4 *:w-24 *:text-center *:border-blue-400"
    >
      <NuxtLink to="/libras">LIBRAS</NuxtLink>
      <NuxtLink to="/asl">ASL</NuxtLink>
    </div>
  </div>
</template>

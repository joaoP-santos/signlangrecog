<template>
  <nav class="w-full bg-primary md:absolute top-0 left-0 z-50">
    <div class="container mx-auto px-4 py-4 flex items-center justify-between">
      <!-- Brand Name -->
      <NuxtLink to="/" class="w-8">
        <NuxtImg src="/images/logo.png" />
      </NuxtLink>
      <!-- Navigation Links -->
      <div class="flex text-sm items-center gap-8">
        <NuxtLink to="/login">Entrar</NuxtLink>
        <NuxtLink
          to="/signup"
          class="bg-blue px-5 py-3 text-bright rounded-[4px]"
          >Comece</NuxtLink
        >
      </div>
    </div>
  </nav>
</template>

<script setup>
// No script needed
</script>

<style scoped>
/* Custom styles if needed */
</style>

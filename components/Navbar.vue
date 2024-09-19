<template>
    <nav class="w-full bg-black fixed top-0 left-0 z-50">
      <div class="container mx-auto px-4 py-4 flex items-center justify-between">
        <!-- Brand Name -->
        <NuxtLink to="/" class="text-xl font-semibold text-white">
          Sign Language Recognition
        </NuxtLink>
        <!-- Navigation Links -->
        <div class="flex items-center space-x-6">
          <!-- About Us Link -->
          <NuxtLink to="/about" class="text-white hover:text-gray-300">
            About Us
          </NuxtLink>
          <!-- Social Media Icons -->
          <div class="flex space-x-4">
            <a
              href="https://facebook.com"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Facebook"
              class="text-white hover:text-gray-300"
            >
              <FontAwesomeIcon :icon="['fab', 'facebook-f']" />
            </a>
            <a
              href="https://twitter.com"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Twitter"
              class="text-white hover:text-gray-300"
            >
              <FontAwesomeIcon :icon="['fab', 'twitter']" />
            </a>
            <a
              href="https://instagram.com"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Instagram"
              class="text-white hover:text-gray-300"
            >
              <FontAwesomeIcon :icon="['fab', 'instagram']" />
            </a>
          </div>
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
  
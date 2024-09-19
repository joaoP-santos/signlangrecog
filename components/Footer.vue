<template>
    <footer class="bg-black text-white py-6">
      <div class="container mx-auto px-4 text-center">
        <p class="text-sm">&copy; {{ new Date().getFullYear() }} Sign Language Recognition. All rights reserved.</p>
        <div class="flex justify-center space-x-4 mt-4">
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
    </footer>
  </template>
  
  <script setup>
  // No script needed
  </script>
  
  <style scoped>
  /* Custom styles if needed */
  </style>
  
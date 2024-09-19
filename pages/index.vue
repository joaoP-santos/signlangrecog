<!-- pages/index.vue -->
<script setup>
import Footer from '~/components/Footer.vue';
</script>

<template>
  <div>
    <!-- Hero Section -->
    <section class="bg-white text-black pt-12 pb-12">
      <div
        class="container mx-auto px-4 flex flex-col md:flex-row items-center md:items-center md:h-screen"
      >
        <!-- Left Side: Text and Buttons -->
        <div class="w-full md:w-1/2 md:pr-8 text-center md:text-left">
          <h1 class="text-6xl font-light mb-6">
            Let's start LIVE sign language recognition
          </h1>
          <p class="text-xl mb-8">
            But first, pick your language
          </p>
          <div class="flex justify-center md:justify-start space-x-4">
            <NuxtLink
              to="/libras"
              class="px-8 py-4 border border-black text-black font-medium rounded hover:bg-black hover:text-white transition duration-300"
            >
              LIBRAS
            </NuxtLink>
            <NuxtLink
              to="/asl"
              class="px-8 py-4 border border-black text-black font-medium rounded hover:bg-black hover:text-white transition duration-300"
            >
              ASL
            </NuxtLink>
          </div>
        </div>

        <!-- Right Side: Image -->
        <div class="w-full md:w-1/2 mt-8 md:mt-0">
          <img
            src="../public/icons/hero-image.png"
            alt="Sign Language Illustration"
            class="w-full h-auto"
          />
        </div>
      </div>
    </section>

    <!-- Body Section -->
    <section class="bg-white text-black py-12">
      <div class="container mx-auto px-4">
        <h2 class="text-4xl font-semibold mb-6">Welcome to Our App</h2>
        <p class="text-lg mb-4">
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua.
        </p>
        <p class="text-lg mb-4">
          Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris
          nisi ut aliquip ex ea commodo consequat.
        </p>
        <!-- Add more dummy text or content as needed -->
      </div>
    </section>

    <!-- Footer -->
    <Footer />
  </div>
</template>

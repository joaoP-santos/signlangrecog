<!-- pages/index.vue -->
<script setup>
import Footer from "~/components/Footer.vue";
</script>

<template>
  <Navbar />
  <!-- Seção Hero -->
  <section class="pt-12">
    <div
      class="container mx-auto px-4 flex flex-col md:flex-row items-center md:items-center justify-between md:h-[90vh]"
    >
      <!-- Lado Esquerdo: Texto e Botões -->
      <div class="w-full md:w-1/2 md:pr-8 text-center md:text-left">
        <h1 class="text-6xl font-bold mb-6">Pratique língua de sinais.</h1>
        <p class="text-xl mb-8">Mas primeiro, escolha seu idioma:</p>
        <div class="flex justify-center md:justify-start space-x-4">
          <NuxtLink to="/libras">
            <BorderlessButton text="LIBRAS" />
          </NuxtLink>
          <NuxtLink to="/asl"> <BorderlessButton text="ASL" /> </NuxtLink>
        </div>
      </div>

      <!-- Lado Direito: Imagem -->
      <div class="w-full md:w-1/2 mt-8 md:mt-0 justify-center">
        <NuxtImg
          src="../static/images/sl-home.png"
          alt="Ilustração de Linguagem de Sinais"
          class="w-auto h-full m-auto max-h-[70vh]"
        />
      </div>
    </div>
  </section>

  <!-- Seção Principal -->
  <section class="py-12">
    <div class="container mx-auto px-4">
      <h2 class="text-4xl font-semibold mb-6">
        Bem-vindo ao nosso aplicativo.
      </h2>
      <p class="text-md mb-4">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
        tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
        veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
        commodo consequat.
      </p>
    </div>
  </section>

  <!-- Rodapé -->
  <Footer />
</template>

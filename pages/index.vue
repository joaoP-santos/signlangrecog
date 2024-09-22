<!-- pages/index.vue -->
<script setup>
import Footer from '~/components/Footer.vue';
</script>

<template>
  <div>
    <!-- Seção Hero -->
    <section class="bg-white text-black pt-12 pb-12">
      <div
        class="container mx-auto px-4 flex flex-col md:flex-row items-center md:items-center md:h-screen"
      >
        <!-- Lado Esquerdo: Texto e Botões -->
        <div class="w-full md:w-1/2 md:pr-8 text-center md:text-left">
          <h1 class="text-6xl font-light mb-6">
            Vamos iniciar o reconhecimento de linguagem de sinais AO VIVO
          </h1>
          <p class="text-xl mb-8">
            Mas primeiro, escolha seu idioma
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

        <!-- Lado Direito: Imagem -->
        <div class="w-full md:w-1/2 mt-8 md:mt-0">
          <img
            src="../public/icons/hero-image.png"
            alt="Ilustração de Linguagem de Sinais"
            class="w-full h-auto"
          />
        </div>
      </div>
    </section>

    <!-- Seção Principal -->
    <section class="bg-white text-black py-12">
      <div class="container mx-auto px-4">
        <h2 class="text-4xl font-semibold mb-6">Bem-vindo ao Nosso Aplicativo</h2>
        <p class="text-lg mb-4">
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua.
        </p>
        <p class="text-lg mb-4">
          Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris
          nisi ut aliquip ex ea commodo consequat.
        </p>
        <!-- Adicione mais texto ou conteúdo conforme necessário -->
      </div>
    </section>

    <!-- Rodapé -->
    <Footer />
  </div>
</template>

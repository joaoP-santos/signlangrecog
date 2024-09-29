<!-- components/Bar.vue -->
<template>
  <div class="bar-container">
    <div class="bar-fill" :style="{ width: width + '%' }"></div>
  </div>
</template>

<script setup>
defineProps({
  width: {
    type: Number,
    required: true,
  },
});
</script>

<style scoped>
.bar-container {
  width: 100%;
  background-color: #e0e0e0;
  border-radius: 5px;
  overflow: hidden;
  height: 20px;
  margin-top: 5px;
}

.bar-fill {
  height: 100%;
  background-color: #4caf50; /* Blue color */
  width: 0%;
  transition: width 0.01s linear;
}
</style>

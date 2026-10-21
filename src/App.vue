<script setup lang="ts">
import CloudBackground from './components/CloudBackground.vue';
</script>

<template>
  <main class="home">
    <CloudBackground />
  </main>
</template>

<style scoped>
.home {
  position: relative;
  isolation: isolate;
  min-height: 100svh;
}
</style>

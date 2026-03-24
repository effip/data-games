<script setup lang="ts">
import { ref } from 'vue';

const particlesOptions = ref({
  fpsLimit: 120,
  particles: {
    color: {
      value: "#ffffff"
    },
    move: {
      direction: "none",
      enable: true,
      outModes: "out",
      random: false,
      speed: 0.2,
      straight: false
    },
    number: {
      density: {
        enable: true,
        area: 800
      },
      value: 200
    },
    opacity: {
      value: { min: 0.1, max: 0.8 },
      animation: {
        enable: true,
        speed: 1,
        sync: false
      }
    },
    shape: {
      type: "square"
    },
    size: {
      value: { min: 0.3, max: 2 }
    }
  },
  detectRetina: true
});
</script>

<template>
  <div class="background-container">
    <vue-particles
      id="tsparticles"
      :options="particlesOptions"
    />
  </div>
</template>

<style scoped>
.background-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1; /* Ensure it stays behind other content */
  pointer-events: none; /* Let clicks pass through */
}
</style>

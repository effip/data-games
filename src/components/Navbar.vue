<script setup lang="ts">
import { RouterLink } from 'vue-router'
import Logo from './Logo.vue'
</script>

<template>
  <nav class="navbar">
    <RouterLink to="/" class="brand">
      <Logo class="logo" />
      <span class="title">data:gаmеs</span>
    </RouterLink>

    <div class="nav-links">
      <RouterLink to="/g" class="nav-link">Gаmеs</RouterLink>
      <RouterLink to="/random" class="nav-link">Random</RouterLink>
    </div>
  </nav>
</template>

<style scoped>
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 80px;
  background: linear-gradient(to bottom, #202124 0%, transparent 100%);
  display: flex;
  align-items: flex-start;
  padding: 16px 24px 0 24px;
  box-sizing: border-box;
  z-index: 1000;
  pointer-events: none;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
  pointer-events: auto;
  cursor: pointer;
  text-decoration: none;
}

.logo {
  height: 32px;
  width: auto;
}

.title {
  color: #9aa0a6;
  font-size: 22px;
  font-weight: 500;
  letter-spacing: -0.3px;
}

.nav-links {
  display: flex;
  gap: 24px;
  pointer-events: auto;
  align-items: center;
  height: 32px;
  margin-left: auto;
}

.nav-link {
  color: #9aa0a6;
  text-decoration: none;
  font-size: 16px;
  font-weight: 500;
  transition: color 0.2s ease;
}

.nav-link:hover {
  color: #e8eaed;
}

.nav-link.router-link-active {
  color: #ffffff;
}
</style>

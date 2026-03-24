<script setup lang="ts">
import { RouterLink } from 'vue-router'
import { ref, onMounted } from 'vue'

const gameCount = ref(0)
const displayCount = ref(0)

const zonesUrls = [
  'https://cdn.jsdelivr.net/gh/gn-math/assets@main/zones.json',
  'https://cdn.jsdelivr.net/gh/gn-math/assets@latest/zones.json',
  'https://cdn.jsdelivr.net/gh/gn-math/assets@master/zones.json',
  'https://cdn.jsdelivr.net/gh/gn-math/assets/zones.json',
]
let zonesURL = zonesUrls[Math.floor(Math.random() * zonesUrls.length)]

const easeOutExpo = (x: number): number => {
  return x === 1 ? 1 : 1 - Math.pow(2, -10 * x)
}

const animateCount = (target: number, duration: number = 2000) => {
  const start = 0
  const startTime = performance.now()

  const frame = (currentTime: number) => {
    const elapsed = currentTime - startTime
    const progress = Math.min(elapsed / duration, 1)
    const ease = easeOutExpo(progress)
    
    displayCount.value = Math.floor(start + (target - start) * ease)

    if (progress < 1) {
      requestAnimationFrame(frame)
    } else {
      displayCount.value = target
    }
  }

  requestAnimationFrame(frame)
}

const fetchGameCount = async () => {
  try {
    // Attempt to get latest commit SHA (simplified from Math.vue)
    let sha = ''
    try {
      const shaResponse = await fetch(
        `https://api.github.com/repos/gn-math/assets/commits?t=${Date.now()}`,
      )
      if (shaResponse.ok) {
        const shaJson = await shaResponse.json()
        sha = shaJson[0]?.sha
      }
    } catch (e) {
      /* ignore */
    }

    if (!sha) {
      try {
        const secondaryResponse = await fetch(
          `https://raw.githubusercontent.com/gn-math/xml/refs/heads/main/sha.txt?t=${Date.now()}`,
        )
        if (secondaryResponse.ok) {
          sha = (await secondaryResponse.text()).trim()
        }
      } catch (e) {
        /* ignore */
      }
    }

    if (sha) {
      zonesURL = `https://cdn.jsdelivr.net/gh/gn-math/assets@${sha}/zones.json`
    }

    const response = await fetch(`${zonesURL}?t=${Date.now()}`)
    const json = await response.json()
    if (Array.isArray(json)) {
      gameCount.value = json.length
      animateCount(gameCount.value)
    }
  } catch (error) {
    console.error('Failed to fetch game count', error)
    // Fallback animation to something reasonable if fetch fails? 
    // Or just stay at 0/1. Let's assume fetch works or stays 0.
  }
}

onMounted(() => {
  fetchGameCount()
})
</script>

<template>
  <div class="home-container">
    <div class="floating-card">
      <h1 class="title">data:<span class="low-opac">//</span>home</h1>
      <p class="subtitle">We have <span class="bold">{{ displayCount }}</span> gаmеs to enjoy.</p>

      <div class="actions">
        <RouterLink to="/g" class="cta-button primary">
          Plаy gаmеs
        </RouterLink>
      </div>
    </div>
  </div>
</template>

<style scoped>
.home-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  box-sizing: border-box;
}

.floating-card {
  padding: 60px 40px;
  text-align: center;
  max-width: 500px;
  width: 100%;
}

.title {
  font-size: 3.5rem;
  font-weight: 700; /* Google Sans Bold */
  margin: 0 0 16px 0;
  background: linear-gradient(135deg, #ffffff 0%, #e8eaed 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: -1px;
}

.low-opac {
  opacity: 0.5;
}

.bold {
  font-weight: bold;
  color: #9aa0a6;
}

.subtitle {
  font-size: 1.25rem;
  color: #9aa0a6;
  margin: 0 0 40px 0;
  font-weight: 400;
  line-height: 1.6;
}

.actions {
  display: flex;
  justify-content: center;
  gap: 16px;
  flex-wrap: wrap;
}

.cta-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 12px 28px;
  border-radius: 24px;
  text-decoration: none;
  font-weight: 500;
  font-size: 1rem;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  outline-offset: 0;
}

.cta-button.primary {
  background: #8ab4f8;
  color: #202124;
}

.cta-button.primary:hover {
  background: #aecbfa;
  outline: 2px solid #aecbfa;
  outline-offset: 2px;
}

/* Responsive adjustments */
@media (max-width: 600px) {
  .title {
    font-size: 2.5rem;
  }

  .floating-card {
    padding: 40px 24px;
  }

  .actions {
    flex-direction: column;
    width: 100%;
  }

  .cta-button {
    width: 100%;
    box-sizing: border-box;
  }
}
</style>

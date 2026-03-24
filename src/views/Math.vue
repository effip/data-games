<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import IconSearch from '~icons/ic/round-search'
import IconRefresh from '~icons/ic/round-refresh'

const router = useRouter()

// --- State ---
const zones = ref<any[]>([])
const loading = ref(true)
const searchQuery = ref('')
const sortBy = ref('name')
const filterBy = ref('none')
const allTags = ref<string[]>([])
const popularityData = ref<Record<string, Record<string, number>>>({
  year: {},
  month: {},
  week: {},
  day: {},
})

// --- Constants ---
const zonesUrls = [
  'https://cdn.jsdelivr.net/gh/gn-math/assets@main/zones.json',
  'https://cdn.jsdelivr.net/gh/gn-math/assets@latest/zones.json',
  'https://cdn.jsdelivr.net/gh/gn-math/assets@master/zones.json',
  'https://cdn.jsdelivr.net/gh/gn-math/assets/zones.json',
]
let zonesURL = zonesUrls[Math.floor(Math.random() * zonesUrls.length)]
const coverURL = 'https://cdn.jsdelivr.net/gh/gn-math/covers@main'
const htmlURL = 'https://cdn.jsdelivr.net/gh/gn-math/html@main'

// --- Methods ---
const toTitleCase = (str: string) => {
  return str.replace(
    /\w\S*/g,
    (text) => text.charAt(0).toUpperCase() + text.substring(1).toLowerCase(),
  )
}

const resolveUrl = (url: string) => {
  return url.replace('{COVER_URL}', coverURL).replace('{HTML_URL}', htmlURL)
}

const fetchPopularity = async (duration: string) => {
  try {
    const response = await fetch(
      `https://data.jsdelivr.com/v1/stats/packages/gh/gn-math/html@main/files?period=${duration}`,
    )
    const data = await response.json()
    data.forEach((file: any) => {
      const idMatch = file.name.match(/\/(\d+)\.html$/)
      if (idMatch) {
        const id = parseInt(idMatch[1])
        if (!popularityData.value[duration]) popularityData.value[duration] = {}
        popularityData.value[duration][id] = file.hits?.total ?? 0
      }
    })
  } catch (error) {
    console.warn(`Failed to fetch popularity for ${duration}`, error)
  }
}

const listZones = async () => {
  loading.value = true
  try {
    // Attempt to get latest commit SHA
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
    zones.value = json
    // Discord is always featured (index 0 logic from source)
    if (zones.value.length > 0) zones.value[0].featured = true

    // Extract tags
    const tags = new Set<string>()
    for (const zone of zones.value) {
      if (Array.isArray(zone.special)) {
        zone.special.forEach((tag: string) => tags.add(tag))
      }
    }
    allTags.value = Array.from(tags).sort()

    // Fetch popularity in background
    Promise.all([
      fetchPopularity('year'),
      fetchPopularity('month'),
      fetchPopularity('week'),
      fetchPopularity('day'),
    ])
  } catch (error) {
    console.error('Error loading zones:', error)
  } finally {
    loading.value = false
  }
}

const openZone = (zone: any) => {
  router.push({ path: '/p', query: { id: zone.id } })
}

// --- Computed ---
const sortedZones = computed(() => {
  let result = [...zones.value]

  // Sort
  if (sortBy.value === 'name') {
    result.sort((a, b) => a.name.localeCompare(b.name))
  } else if (sortBy.value === 'id') {
    result.sort((a, b) => a.id - b.id)
  } else if (sortBy.value === 'popular') {
    result.sort(
      (a, b) =>
        (popularityData.value['year']?.[b.id] ?? 0) - (popularityData.value['year']?.[a.id] ?? 0),
    )
  } else if (sortBy.value === 'trendingMonth') {
    result.sort(
      (a, b) =>
        (popularityData.value['month']?.[b.id] ?? 0) - (popularityData.value['month']?.[a.id] ?? 0),
    )
  } else if (sortBy.value === 'trendingWeek') {
    result.sort(
      (a, b) =>
        (popularityData.value['week']?.[b.id] ?? 0) - (popularityData.value['week']?.[a.id] ?? 0),
    )
  } else if (sortBy.value === 'trendingDay') {
    result.sort(
      (a, b) =>
        (popularityData.value['day']?.[b.id] ?? 0) - (popularityData.value['day']?.[a.id] ?? 0),
    )
  }

  return result
})

const processedZones = computed(() => {
  let result = sortedZones.value

  // Filter by search
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter((z) => z.name.toLowerCase().includes(query))
  }

  // Filter by tag
  if (filterBy.value !== 'none') {
    result = result.filter((z) => z.special?.includes(filterBy.value))
  }

  return result
})

const featuredZones = computed(() => {
  if (searchQuery.value || filterBy.value !== 'none') return []
  return sortedZones.value.filter((z) => z.featured)
})

const displayZonesList = computed(() => {
  return processedZones.value
})

onMounted(() => {
  listZones()
})
</script>

<template>
  <div class="math-container">
    <div class="content-wrapper">
      <header class="header">
        <h1 class="page-title">data:<span class="low-opac">//</span>gаmеs</h1>

        <div class="controls-row">
          <div class="search-bar">
            <IconSearch class="search-icon" style="font-size: 1.25rem" />
            <input v-model="searchQuery" type="text" placeholder="Search..." />
          </div>

          <div class="filters">
            <select v-model="sortBy">
              <option value="name">Name</option>
              <option value="id">Date Added</option>
              <option value="popular">Popular</option>
              <option value="trendingDay">Trending (Day)</option>
              <option value="trendingWeek">Trending (Week)</option>
              <option value="trendingMonth">Trending (Month)</option>
            </select>

            <select v-model="filterBy">
              <option value="none">All Tags</option>
              <option v-for="tag in allTags" :key="tag" :value="tag">
                {{ toTitleCase(tag) }}
              </option>
            </select>

            <button class="icon-btn" @click="listZones" title="Refresh">
              <IconRefresh style="font-size: 1.25rem" />
            </button>
          </div>
        </div>
      </header>

      <main class="main-content">
        <div v-if="loading" class="loading">
          <div class="spinner"></div>
        </div>

        <template v-else>
          <!-- Featured Zones -->
          <div v-if="featuredZones.length > 0" class="section">
            <h2 class="section-title">Featured</h2>
            <div class="zones-grid">
              <div
                v-for="zone in featuredZones"
                :key="zone.id"
                class="zone-card featured"
                @click="openZone(zone)"
              >
                <div class="zone-image-wrapper">
                  <img :src="resolveUrl(zone.cover)" :alt="zone.name" loading="lazy" />
                </div>
                <div class="zone-info">
                  <span class="zone-name">{{ zone.name }}</span>
                </div>
              </div>
            </div>
          </div>

          <!-- All Zones -->
          <div class="section">
            <h2 class="section-title">All Gаmеѕ ({{ displayZonesList.length }})</h2>
            <div v-if="displayZonesList.length === 0" class="no-results">No gаmеѕ found.</div>
            <div class="zones-grid">
              <div
                v-for="zone in displayZonesList"
                :key="zone.id"
                class="zone-card"
                @click="openZone(zone)"
              >
                <div class="zone-image-wrapper">
                  <img :src="resolveUrl(zone.cover)" :alt="zone.name" loading="lazy" />
                </div>
                <div class="zone-info">
                  <span class="zone-name">{{ zone.name }}</span>
                </div>
              </div>
            </div>
          </div>
        </template>
      </main>
    </div>
  </div>
</template>

<style scoped>
.math-container {
  width: 100%;
  min-height: 100vh;
  padding: 80px 20px 20px;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
}

.content-wrapper {
  max-width: 1200px;
  width: 100%;
}

.header {
  margin-bottom: 2rem;
}

.page-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0 0 1.5rem 0;
  background: linear-gradient(135deg, #ffffff 0%, #e8eaed 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: -1px;
}

.low-opac {
  opacity: 0.5;
}

.controls-row {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  align-items: center;
  background: rgba(32, 33, 36, 0.6);
  padding: 1rem;
  border-radius: 16px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.search-bar {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  background: rgba(255, 255, 255, 0.05);
  padding: 0.5rem 1rem;
  border-radius: 24px;
  flex: 1;
  min-width: 200px;
  transition: background 0.2s;
}

.search-bar:focus-within {
  background: rgba(255, 255, 255, 0.1);
}

.search-icon {
  color: #9aa0a6;
}

.search-bar input {
  background: transparent;
  border: none;
  color: #e8eaed;
  width: 100%;
  outline: none;
  font-size: 1rem;
}

.filters {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

select {
  background: rgba(255, 255, 255, 0.05);
  color: #e8eaed;
  border: 1px solid transparent;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  outline: none;
  cursor: pointer;
  transition: all 0.2s;
}

select:hover {
  background: rgba(255, 255, 255, 0.1);
}

.icon-btn {
  background: transparent;
  border: none;
  color: #9aa0a6;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.icon-btn:hover {
  color: #e8eaed;
  background: rgba(255, 255, 255, 0.1);
}

.section {
  margin-bottom: 3rem;
}

.section-title {
  font-size: 1.25rem;
  color: #e8eaed;
  margin-bottom: 1rem;
  font-weight: 500;
}

.zones-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 1.5rem;
}

.zone-card {
  background: #303134;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  border: 1px solid transparent;
}

.zone-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  border-color: rgba(138, 180, 248, 0.5);
}

.zone-image-wrapper {
  width: 100%;
  aspect-ratio: 1;
  background: #202124;
  overflow: hidden;
}

.zone-image-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.zone-card:hover .zone-image-wrapper img {
  transform: scale(1.05);
}

.zone-info {
  padding: 1rem;
}

.zone-name {
  font-size: 0.9rem;
  color: #e8eaed;
  font-weight: 500;
  display: block;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.loading, .no-results {
  text-align: center;
  padding: 4rem;
  color: #9aa0a6;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid rgba(255, 255, 255, 0.1);
  border-top-color: #8ab4f8;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

@media (max-width: 768px) {
  .controls-row {
    flex-direction: column;
    align-items: stretch;
  }

  .filters {
    justify-content: space-between;
  }
}
</style>

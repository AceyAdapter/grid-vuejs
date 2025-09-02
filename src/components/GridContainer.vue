<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

const screenWidth = ref(window.innerWidth)
const containerRef = ref<HTMLElement>()

const isMobile = computed(() => screenWidth.value < 768)
const columns = computed(() => (isMobile.value ? 4 : 8))
const rows = computed(() => 6)

// Grid template CSS string
const gridTemplate = computed(() => {
  return `repeat(${columns.value}, 1fr)`
})

// Handle window resize
const handleResize = () => {
  screenWidth.value = window.innerWidth
}

// Lifecycle hooks
onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})

// Generate grid cells for visualization
const gridCells = computed(() => {
  const total = columns.value * rows.value
  return Array.from({ length: total }, (_, index) => ({
    id: index,
    x: index % columns.value,
    y: Math.floor(index / columns.value),
  }))
})
</script>

<template>
  <div class="grid-container">
    <!-- Grid wrapper with responsive sizing -->
    <div
      ref="containerRef"
      class="grid-wrapper"
      :style="{
        gridTemplateColumns: gridTemplate,
        gridTemplateRows: `repeat(${rows}, 1fr)`,
      }"
    >
      <!-- Grid cells for visualization (you can remove these later) -->
      <div
        v-for="cell in gridCells"
        :key="cell.id"
        class="grid-cell"
        :data-x="cell.x"
        :data-y="cell.y"
      >
        {{ cell.x }},{{ cell.y }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.grid-container {
  width: 100%;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.grid-wrapper {
  display: grid;
  gap: 8px;
  width: 100%;
  aspect-ratio: 4/3;
  background-color: #2a2a2a;
  border-radius: 12px;
  padding: 8px;
  border: 2px solid #3a3a3a;
}

.grid-cell {
  background-color: #404040;
  border: 1px solid #555;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #ccc;
  transition: all 0.2s ease;
  min-height: 40px;
}

.grid-cell:hover {
  background-color: #505050;
  border-color: #666;
}

.grid-info {
  margin-top: 16px;
  text-align: center;
  font-size: 14px;
}

.grid-info p {
  margin: 4px 0;
}

/* Responsive adjustments */
@media (max-width: 767px) {
  .grid-wrapper {
    aspect-ratio: 2/3; /* Taller on mobile */
  }

  .grid-cell {
    font-size: 10px;
    min-height: 30px;
  }
}

@media (min-width: 768px) {
  .grid-wrapper {
    max-width: 800px;
    aspect-ratio: 4/3; /* Wider on desktop */
  }
}
</style>

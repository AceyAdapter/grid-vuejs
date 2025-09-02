<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import FloatingAddButton from './FloatingAddButton.vue'
import CreateWidgetModal from './CreateWidgetModal.vue'
import WidgetObject from './WidgetObject.vue'

// Widget interface
interface WidgetData {
  id: string
  type: 'text' | 'image'
  position: { x: number; y: number }
  size: { width: number; height: number }
  content: unknown
  createdAt: number
}

const screenWidth = ref(window.innerWidth)
const containerRef = ref<HTMLElement>()
const isModalOpen = ref(false)
const widgets = ref<WidgetData[]>([])

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

// Simple auto-placement algorithm
const findAvailablePosition = (width: number, height: number) => {
  // Create a map of occupied cells
  const occupiedCells = new Set<string>()

  widgets.value.forEach((widget) => {
    for (let x = widget.position.x; x < widget.position.x + widget.size.width; x++) {
      for (let y = widget.position.y; y < widget.position.y + widget.size.height; y++) {
        occupiedCells.add(`${x},${y}`)
      }
    }
  })

  // Find first available position
  for (let y = 0; y <= rows.value - height; y++) {
    for (let x = 0; x <= columns.value - width; x++) {
      let canPlace = true

      // Check if all cells in this position are available
      for (let checkX = x; checkX < x + width; checkX++) {
        for (let checkY = y; checkY < y + height; checkY++) {
          if (occupiedCells.has(`${checkX},${checkY}`)) {
            canPlace = false
            break
          }
        }
        if (!canPlace) break
      }

      if (canPlace) {
        return { x, y }
      }
    }
  }

  return null // No space available
}

// Generate unique ID
const generateId = () => {
  return `widget_${Date.now()}_${Math.random().toString(36)}`
}

// Modal handlers
const openModal = () => {
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const createWidget = ({
  type,
  width,
  height,
}: {
  type: 'text' | 'image'
  width: number
  height: number
}) => {
  const position = findAvailablePosition(width, height)

  if (!position) {
    alert('No space available for this widget size!')
    return
  }

  const newWidget: WidgetData = {
    id: generateId(),
    type,
    position,
    size: { width, height },
    content: {},
    createdAt: Date.now(),
  }

  widgets.value.push(newWidget)
}

// Generate grid cells for empty spaces (optional visualization)
const gridCells = computed(() => {
  const total = columns.value * rows.value
  const occupiedCells = new Set<string>()

  // Mark occupied cells
  widgets.value.forEach((widget) => {
    for (let x = widget.position.x; x < widget.position.x + widget.size.width; x++) {
      for (let y = widget.position.y; y < widget.position.y + widget.size.height; y++) {
        occupiedCells.add(`${x},${y}`)
      }
    }
  })

  return Array.from({ length: total }, (_, index) => {
    const x = index % columns.value
    const y = Math.floor(index / columns.value)
    const key = `${x},${y}`

    return {
      id: index,
      x,
      y,
      occupied: occupiedCells.has(key),
    }
  })
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
      <!-- Grid cells for empty spaces -->
      <div
        v-for="cell in gridCells"
        :key="cell.id"
        v-show="!cell.occupied"
        class="grid-cell"
        :data-x="cell.x"
        :data-y="cell.y"
      ></div>

      <!-- Widgets -->
      <WidgetObject v-for="widget in widgets" :key="widget.id" :widget="widget" />
    </div>

    <!-- Floating Add Button -->
    <FloatingAddButton @open-modal="openModal" />

    <!-- Create Widget Modal -->
    <CreateWidgetModal
      :is-open="isModalOpen"
      :columns="columns"
      @close="closeModal"
      @create-widget="createWidget"
    />
  </div>
</template>

<style scoped>
.grid-container {
  width: 100%;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
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
  color: #666;
  transition: all 0.2s ease;
  min-height: 40px;
  opacity: 0.3;
}

.grid-cell:hover {
  background-color: #505050;
  border-color: #666;
  opacity: 0.6;
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

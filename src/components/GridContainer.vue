<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import FloatingAddButton from './FloatingAddButton.vue'
import CreateWidgetModal from './CreateWidgetModal.vue'
import WidgetObject from './WidgetObject.vue'
import TrashZone from './TrashZone.vue'

// Updated Widget interface to match new content structure
interface WidgetData {
  id: string
  type: 'text' | 'image'
  position: { x: number; y: number }
  size: { width: number; height: number }
  content: {
    text?: string
    textColor?: string
    backgroundColor?: string
    url?: string
    localFile?: string
  }
  createdAt: number
}

const screenWidth = ref(window.innerWidth)
const containerRef = ref<HTMLElement>()
const isModalOpen = ref(false)
const widgets = ref<WidgetData[]>([])

// Drag state for visual feedback
const isDragging = ref(false)
const draggedWidgetId = ref<string | null>(null)
const hoveredCells = ref<{ x: number; y: number }[]>([])
const isValidDrop = ref(true)

// Trash zone state - centralized here
const isOverTrash = ref(false)

const isMobile = computed(() => screenWidth.value < 768)
const columns = computed(() => (isMobile.value ? 4 : 8))
const rows = computed(() => 6)

// Grid template CSS string
const gridTemplate = computed(() => {
  return `repeat(${columns.value}, 1fr)`
})

// Centralized trash zone detection
const isMouseOverTrashZone = (mouseX: number, mouseY: number): boolean => {
  const windowHeight = window.innerHeight
  const isMobile = window.innerWidth < 768
  const trashZoneHeight = isMobile ? 60 : 80
  const trashZoneWidth = isMobile ? 150 : 200
  const trashZoneBottom = isMobile ? 20 : 24
  const trashZoneLeft = isMobile ? 20 : 24

  const trashZoneTop = windowHeight - trashZoneHeight - trashZoneBottom
  const trashZoneRight = trashZoneLeft + trashZoneWidth

  return (
    mouseX >= trashZoneLeft &&
    mouseX <= trashZoneRight &&
    mouseY >= trashZoneTop &&
    mouseY <= windowHeight - trashZoneBottom
  )
}

// Handle window resize
const handleResize = () => {
  screenWidth.value = window.innerWidth
}

// Global mouse move handler for trash detection
const handleGlobalMouseMove = (event: MouseEvent) => {
  if (!isDragging.value) return

  isOverTrash.value = isMouseOverTrashZone(event.clientX, event.clientY)
}

// Global mouse up handler for final drop detection
const handleGlobalMouseUp = (event: MouseEvent) => {
  if (!isDragging.value || !draggedWidgetId.value) return

  const isDroppedOnTrash = isMouseOverTrashZone(event.clientX, event.clientY)

  if (isDroppedOnTrash) {
    deleteWidget(draggedWidgetId.value)
  }
}

// Global touch handlers for mobile
const handleGlobalTouchMove = (event: TouchEvent) => {
  if (!isDragging.value) return

  const touch = event.touches[0]
  isOverTrash.value = isMouseOverTrashZone(touch.clientX, touch.clientY)
}

const handleGlobalTouchEnd = (event: TouchEvent) => {
  if (!isDragging.value || !draggedWidgetId.value) return

  const touch = event.changedTouches[0]
  const isDroppedOnTrash = isMouseOverTrashZone(touch.clientX, touch.clientY)

  if (isDroppedOnTrash) {
    deleteWidget(draggedWidgetId.value)
  }
}

// Lifecycle hooks
onMounted(() => {
  window.addEventListener('resize', handleResize)
  window.addEventListener('beforeunload', clearLocalStorage)
  window.addEventListener('unload', clearLocalStorage)

  // Global mouse/touch handlers for trash detection
  document.addEventListener('mousemove', handleGlobalMouseMove)
  document.addEventListener('mouseup', handleGlobalMouseUp)
  document.addEventListener('touchmove', handleGlobalTouchMove, { passive: true })
  document.addEventListener('touchend', handleGlobalTouchEnd)

  loadFromLocalStorage()
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  window.removeEventListener('beforeunload', clearLocalStorage)
  window.removeEventListener('unload', clearLocalStorage)

  // Clean up global handlers
  document.removeEventListener('mousemove', handleGlobalMouseMove)
  document.removeEventListener('mouseup', handleGlobalMouseUp)
  document.removeEventListener('touchmove', handleGlobalTouchMove)
  document.removeEventListener('touchend', handleGlobalTouchEnd)

  clearLocalStorage()
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
  return `widget_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`
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
  content,
}: {
  type: 'text' | 'image'
  width: number
  height: number
  content: {
    text?: string
    textColor?: string
    backgroundColor?: string
    url?: string
    localFile?: string
  }
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
    content: content,
    createdAt: Date.now(),
  }

  widgets.value.push(newWidget)
  saveToLocalStorage()
}

// Update widget position after drag and drop
const updateWidget = ({ id, position }: { id: string; position: { x: number; y: number } }) => {
  const widget = widgets.value.find((w) => w.id === id)
  if (widget) {
    widget.position = position
    saveToLocalStorage()
  }
}

// Delete widget function
const deleteWidget = (id: string) => {
  const index = widgets.value.findIndex((w) => w.id === id)
  if (index !== -1) {
    widgets.value.splice(index, 1)
    saveToLocalStorage()
  }

  resetDragState()
}

// Centralized function to reset drag state
const resetDragState = () => {
  isDragging.value = false
  draggedWidgetId.value = null
  hoveredCells.value = []
  isValidDrop.value = true
  isOverTrash.value = false
}

// Simplified drag event handlers
const handleDragStart = ({ id }: { id: string }) => {
  isDragging.value = true
  draggedWidgetId.value = id
}

const handleDragMove = ({
  hoveredCells: cells,
  isValid,
}: {
  hoveredCells: { x: number; y: number }[]
  isValid: boolean
}) => {
  hoveredCells.value = cells
  isValidDrop.value = isValid
}

const handleDragEnd = () => {
  // Reset all drag states - trash deletion is handled by global handlers
  resetDragState()
}

// LocalStorage functions
const saveToLocalStorage = () => {
  try {
    const data = {
      version: '1.0.0',
      widgets: widgets.value,
      lastUpdated: Date.now(),
    }
    localStorage.setItem('grid-dashboard', JSON.stringify(data))
  } catch (error) {
    console.error('Failed to save to localStorage:', error)
  }
}

const loadFromLocalStorage = () => {
  try {
    const data = localStorage.getItem('grid-dashboard')
    if (data) {
      const parsed = JSON.parse(data)
      if (parsed.widgets && Array.isArray(parsed.widgets)) {
        widgets.value = parsed.widgets
      }
    }
  } catch (error) {
    console.error('Failed to load from localStorage:', error)
  }
}

const clearLocalStorage = () => {
  try {
    localStorage.removeItem('grid-dashboard')
  } catch (error) {
    console.error('Failed to clear localStorage:', error)
  }
}

// Generate grid cells for visualization
const gridCells = computed(() => {
  const total = columns.value * rows.value
  const occupiedCells = new Set<string>()

  // Mark occupied cells (excluding dragged widget)
  widgets.value.forEach((widget) => {
    if (isDragging.value && widget.id === draggedWidgetId.value) {
      return // Skip the dragged widget
    }

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

    // Check if this cell is being hovered during drag
    const isHovered =
      isDragging.value && hoveredCells.value.some((cell) => cell.x === x && cell.y === y)

    return {
      id: index,
      x,
      y,
      occupied: occupiedCells.has(key),
      isHovered,
      isValidHover: isHovered && isValidDrop.value,
      isInvalidHover: isHovered && !isValidDrop.value,
    }
  })
})

// Determine which cells to show based on drag state
const visibleCells = computed(() => {
  if (isDragging.value) {
    // During drag, show all cells for visual feedback
    return gridCells.value
  } else {
    // When not dragging, only show empty cells
    return gridCells.value.filter((cell) => !cell.occupied)
  }
})
</script>

<template>
  <div class="grid-container">
    <!-- Grid wrapper with responsive sizing -->
    <div
      ref="containerRef"
      class="grid-wrapper"
      :class="{ 'dragging-active': isDragging }"
      :style="{
        gridTemplateColumns: gridTemplate,
        gridTemplateRows: `repeat(${rows}, 1fr)`,
      }"
    >
      <!-- Grid cells for empty spaces and drag feedback -->
      <div
        v-for="cell in visibleCells"
        :key="cell.id"
        v-show="!cell.occupied || cell.isHovered"
        :class="[
          'grid-cell',
          {
            'grid-cell-hovered': cell.isHovered,
            'grid-cell-valid': cell.isValidHover,
            'grid-cell-invalid': cell.isInvalidHover,
          },
        ]"
        :data-x="cell.x"
        :data-y="cell.y"
      ></div>

      <!-- Widgets with drag and drop support -->
      <WidgetObject
        v-for="widget in widgets"
        :key="widget.id"
        :widget="widget"
        :columns="columns"
        :rows="rows"
        :widgets="widgets"
        @update-widget="updateWidget"
        @drag-start="handleDragStart"
        @drag-move="handleDragMove"
        @drag-end="handleDragEnd"
      />
    </div>

    <!-- Trash Zone - Only visible during drag -->
    <TrashZone :is-visible="isDragging" :is-active="isOverTrash" />

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
  transition: border-color 0.2s ease;
}

.grid-wrapper.dragging-active {
  border-color: #4f46e5;
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

/* Drag feedback styles */
.grid-cell-hovered {
  opacity: 0.8;
  transform: scale(0.95);
  z-index: 10;
}

.grid-cell-valid {
  background-color: #22c55e !important;
  border-color: #16a34a !important;
  box-shadow: 0 0 12px rgba(34, 197, 94, 0.4);
}

.grid-cell-invalid {
  background-color: #ef4444 !important;
  border-color: #dc2626 !important;
  box-shadow: 0 0 12px rgba(239, 68, 68, 0.4);
  animation: shake 0.3s ease-in-out;
}

@keyframes shake {
  0%,
  100% {
    transform: scale(0.95) translateX(0);
  }
  25% {
    transform: scale(0.95) translateX(-2px);
  }
  75% {
    transform: scale(0.95) translateX(2px);
  }
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

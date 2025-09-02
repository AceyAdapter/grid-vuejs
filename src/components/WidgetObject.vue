<script setup lang="ts">
import { ref, computed } from 'vue'

interface Widget {
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
}

interface Props {
  widget: Widget
  columns: number
  rows: number
  widgets: Widget[]
}

const props = defineProps<Props>()

// Events this component can emit
const emit = defineEmits<{
  updateWidget: [{ id: string; position: { x: number; y: number } }]
  dragStart: [{ id: string }]
  dragEnd: []
  dragMove: [{ hoveredCells: { x: number; y: number }[]; isValid: boolean }]
}>()

// Reactive state for drag operations
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const clickOffset = ref({ x: 0, y: 0 })
const originalPosition = ref({ x: 0, y: 0 })
const originalDimensions = ref({ width: 0, height: 0 })
const hoveredGridPosition = ref<{ x: number; y: number } | null>(null)

// Current hovered position validity
const currentHoverIsValid = ref(true)

// Computed styles for text widgets
const textWidgetStyle = computed(() => {
  if (props.widget.type !== 'text') return {}

  return {
    backgroundColor: props.widget.content.backgroundColor || '#4f46e5',
    color: props.widget.content.textColor || '#f2faff',
  }
})

// Computed image source
const imageSource = computed(() => {
  if (props.widget.type !== 'image') return ''
  return props.widget.content.localFile || props.widget.content.url || ''
})

// Computed style for positioning during drag
const dragStyle = computed(() => {
  if (!isDragging.value) return {}

  return {
    position: 'fixed' as const,
    left: `${dragOffset.value.x}px`,
    top: `${dragOffset.value.y}px`,
    width: `${originalDimensions.value.width}px`,
    height: `${originalDimensions.value.height}px`,
    zIndex: 1000,
    pointerEvents: 'none' as const,
    opacity: 0.8,
    transform: 'rotate(2deg)',
  }
})

// Check if a position would cause overlap with other widgets
const wouldOverlap = (newX: number, newY: number): boolean => {
  // Check bounds first
  if (
    newX < 0 ||
    newY < 0 ||
    newX + props.widget.size.width > props.columns ||
    newY + props.widget.size.height > props.rows
  ) {
    return true
  }

  // Check overlap with other widgets (excluding current widget)
  return props.widgets.some((otherWidget) => {
    if (otherWidget.id === props.widget.id) return false

    const otherRight = otherWidget.position.x + otherWidget.size.width
    const otherBottom = otherWidget.position.y + otherWidget.size.height
    const newRight = newX + props.widget.size.width
    const newBottom = newY + props.widget.size.height

    return !(
      newX >= otherRight ||
      newRight <= otherWidget.position.x ||
      newY >= otherBottom ||
      newBottom <= otherWidget.position.y
    )
  })
}

// Get all cells that would be occupied by the widget at a given position
const getCellsForPosition = (gridX: number, gridY: number) => {
  const cells = []
  for (let x = gridX; x < gridX + props.widget.size.width; x++) {
    for (let y = gridY; y < gridY + props.widget.size.height; y++) {
      cells.push({ x, y })
    }
  }
  return cells
}

// Improved grid position calculation based on widget center
const getGridPositionFromMouse = (mouseX: number, mouseY: number) => {
  const gridElement = document.querySelector('.grid-wrapper') as HTMLElement
  if (!gridElement) return null

  const rect = gridElement.getBoundingClientRect()

  // Calculate relative position within grid (accounting for padding)
  const relativeX = mouseX - rect.left - 8 // 8px padding
  const relativeY = mouseY - rect.top - 8

  // Calculate cell dimensions including gaps
  const cellWidth = (rect.width - 16 - (props.columns - 1) * 8) / props.columns // 16px total padding, 8px gaps
  const cellHeight = (rect.height - 16 - (props.rows - 1) * 8) / props.rows

  // Find which cell the mouse is over
  const cellX = Math.floor(relativeX / (cellWidth + 8))
  const cellY = Math.floor(relativeY / (cellHeight + 8))

  // Calculate the top-left position to center the widget on the mouse
  const centerOffsetX = Math.floor(props.widget.size.width / 2)
  const centerOffsetY = Math.floor(props.widget.size.height / 2)

  const targetX = cellX - centerOffsetX
  const targetY = cellY - centerOffsetY

  // Clamp to valid bounds
  const clampedX = Math.max(0, Math.min(targetX, props.columns - props.widget.size.width))
  const clampedY = Math.max(0, Math.min(targetY, props.rows - props.widget.size.height))

  return { x: clampedX, y: clampedY }
}

// Mouse event handlers
const handleMouseDown = (event: MouseEvent) => {
  event.preventDefault()

  const element = event.currentTarget as HTMLElement
  const rect = element.getBoundingClientRect()

  originalDimensions.value = {
    width: rect.width,
    height: rect.height,
  }

  clickOffset.value = {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top,
  }

  isDragging.value = true
  originalPosition.value = { ...props.widget.position }

  dragOffset.value = {
    x: event.clientX - clickOffset.value.x,
    y: event.clientY - clickOffset.value.y,
  }

  // Emit drag start event
  emit('dragStart', { id: props.widget.id })

  document.addEventListener('mousemove', handleMouseMove)
  document.addEventListener('mouseup', handleMouseUp)
  document.body.style.userSelect = 'none'
}

const handleMouseMove = (event: MouseEvent) => {
  if (!isDragging.value) return

  // Update visual position
  dragOffset.value = {
    x: event.clientX - clickOffset.value.x,
    y: event.clientY - clickOffset.value.y,
  }

  // Calculate which grid position we're hovering over
  const gridPos = getGridPositionFromMouse(event.clientX, event.clientY)

  if (gridPos) {
    hoveredGridPosition.value = gridPos
    const isValid = !wouldOverlap(gridPos.x, gridPos.y)

    // Update current validity state
    currentHoverIsValid.value = isValid

    // Only emit cells if valid - don't show invalid red squares
    if (isValid) {
      const cells = getCellsForPosition(gridPos.x, gridPos.y)
      emit('dragMove', { hoveredCells: cells, isValid: true })
    } else {
      emit('dragMove', { hoveredCells: [], isValid: false })
    }
  } else {
    // If we can't calculate a valid grid position, hide the widget
    currentHoverIsValid.value = false
    emit('dragMove', { hoveredCells: [], isValid: false })
  }
}

const handleMouseUp = (event: MouseEvent) => {
  if (!isDragging.value) return

  const gridPos = getGridPositionFromMouse(event.clientX, event.clientY)

  if (gridPos && !wouldOverlap(gridPos.x, gridPos.y)) {
    // Valid position - emit update event
    emit('updateWidget', {
      id: props.widget.id,
      position: gridPos,
    })
  }

  resetDrag()
}

const resetDrag = () => {
  isDragging.value = false
  hoveredGridPosition.value = null
  currentHoverIsValid.value = true // Reset to valid state

  // Emit drag end event
  emit('dragEnd')

  document.removeEventListener('mousemove', handleMouseMove)
  document.removeEventListener('mouseup', handleMouseUp)
  document.body.style.userSelect = ''
}

// Touch event handlers for mobile support
const handleTouchStart = (event: TouchEvent) => {
  event.preventDefault()
  const touch = event.touches[0]
  const element = event.currentTarget as HTMLElement
  const rect = element.getBoundingClientRect()

  originalDimensions.value = {
    width: rect.width,
    height: rect.height,
  }

  clickOffset.value = {
    x: touch.clientX - rect.left,
    y: touch.clientY - rect.top,
  }

  isDragging.value = true
  originalPosition.value = { ...props.widget.position }

  dragOffset.value = {
    x: touch.clientX - clickOffset.value.x,
    y: touch.clientY - clickOffset.value.y,
  }

  emit('dragStart', { id: props.widget.id })

  document.addEventListener('touchmove', handleTouchMove, { passive: false })
  document.addEventListener('touchend', handleTouchEnd)
}

const handleTouchMove = (event: TouchEvent) => {
  if (!isDragging.value) return
  event.preventDefault()

  const touch = event.touches[0]

  dragOffset.value = {
    x: touch.clientX - clickOffset.value.x,
    y: touch.clientY - clickOffset.value.y,
  }

  const gridPos = getGridPositionFromMouse(touch.clientX, touch.clientY)

  if (gridPos) {
    hoveredGridPosition.value = gridPos
    const isValid = !wouldOverlap(gridPos.x, gridPos.y)

    // Update current validity state
    currentHoverIsValid.value = isValid

    // Only emit cells if valid - don't show invalid red squares
    if (isValid) {
      const cells = getCellsForPosition(gridPos.x, gridPos.y)
      emit('dragMove', { hoveredCells: cells, isValid: true })
    } else {
      emit('dragMove', { hoveredCells: [], isValid: false })
    }
  } else {
    currentHoverIsValid.value = false
    emit('dragMove', { hoveredCells: [], isValid: false })
  }
}

const handleTouchEnd = (event: TouchEvent) => {
  if (!isDragging.value) return

  const touch = event.changedTouches[0]
  const gridPos = getGridPositionFromMouse(touch.clientX, touch.clientY)

  if (gridPos && !wouldOverlap(gridPos.x, gridPos.y)) {
    emit('updateWidget', {
      id: props.widget.id,
      position: gridPos,
    })
  }

  resetTouchDrag()
}

const resetTouchDrag = () => {
  isDragging.value = false
  hoveredGridPosition.value = null
  currentHoverIsValid.value = true // Reset to valid state

  emit('dragEnd')

  document.removeEventListener('touchmove', handleTouchMove)
  document.removeEventListener('touchend', handleTouchEnd)
}
</script>

<template>
  <div
    class="widget"
    :class="{ 'widget-dragging': isDragging }"
    :style="{
      gridColumn: `${widget.position.x + 1} / span ${widget.size.width}`,
      gridRow: `${widget.position.y + 1} / span ${widget.size.height}`,
      ...dragStyle,
    }"
    @mousedown="handleMouseDown"
    @touchstart="handleTouchStart"
  >
    <div class="widget-content">
      <!-- Drag handle - appears on hover -->
      <div class="drag-handle" v-show="!isDragging">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
          <circle cx="5" cy="12" r="2" fill="currentColor" />
          <circle cx="12" cy="12" r="2" fill="currentColor" />
          <circle cx="19" cy="12" r="2" fill="currentColor" />
          <circle cx="5" cy="5" r="2" fill="currentColor" />
          <circle cx="12" cy="5" r="2" fill="currentColor" />
          <circle cx="19" cy="5" r="2" fill="currentColor" />
          <circle cx="5" cy="19" r="2" fill="currentColor" />
          <circle cx="12" cy="19" r="2" fill="currentColor" />
          <circle cx="19" cy="19" r="2" fill="currentColor" />
        </svg>
      </div>

      <!-- Text Widget -->
      <div v-if="widget.type === 'text'" class="text-widget" :style="textWidgetStyle">
        <p>{{ widget.content.text || 'Text Widget' }}</p>
      </div>

      <!-- Image Widget -->
      <div v-else-if="widget.type === 'image'" class="image-widget">
        <img
          v-if="imageSource"
          :src="imageSource"
          :alt="widget.content.text || 'Image Widget'"
          class="widget-image"
          @error="() => {}"
        />
        <div v-else class="image-placeholder">
          <svg width="32" height="32" viewBox="0 0 24 24" fill="none">
            <rect
              x="3"
              y="3"
              width="18"
              height="18"
              rx="2"
              ry="2"
              stroke="currentColor"
              stroke-width="2"
            />
            <circle cx="8.5" cy="8.5" r="1.5" stroke="currentColor" stroke-width="2" />
            <polyline points="21,15 16,10 5,21" stroke="currentColor" stroke-width="2" />
          </svg>
          <span>No Image</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.widget {
  background-color: #3a3a3a;
  border: 2px solid #555;
  border-radius: 8px;
  overflow: hidden;
  cursor: grab;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.widget:hover {
  border-color: #4f46e5;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.2);
}

.widget-dragging {
  cursor: grabbing;
  border-color: #4f46e5;
  box-shadow: 0 8px 24px rgba(79, 70, 229, 0.3);
}

.widget-content {
  height: 100%;
  width: 100%;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.drag-handle {
  position: absolute;
  top: 8px;
  right: 8px;
  color: #666;
  opacity: 0;
  transition: opacity 0.2s ease;
  pointer-events: none;
  z-index: 10;
}

.widget:hover .drag-handle {
  opacity: 1;
}

.text-widget {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 12px;
  border-radius: 6px;
  word-wrap: break-word;
  overflow: hidden;
}

.text-widget p {
  margin: 0;
  font-size: 1rem;
  line-height: 1.4;
  overflow: hidden;
  text-overflow: ellipsis;
}

.image-widget {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.widget-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 4px;
}

.image-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #666;
  text-align: center;
}

.image-placeholder span {
  font-size: 0.875rem;
}

/* Responsive text sizing */
@media (max-width: 767px) {
  .text-widget p {
    font-size: 0.875rem;
  }

  .drag-handle {
    top: 4px;
    right: 4px;
  }

  .text-widget {
    padding: 8px;
  }
}
</style>

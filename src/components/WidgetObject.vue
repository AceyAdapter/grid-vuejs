<script setup lang="ts">
import { ref, computed } from 'vue'

interface Widget {
  id: string
  type: 'text' | 'image'
  position: { x: number; y: number }
  size: { width: number; height: number }
  content: unknown
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
}>()

// Reactive state for drag operations
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const clickOffset = ref({ x: 0, y: 0 }) // Where in the widget we clicked
const originalPosition = ref({ x: 0, y: 0 })
const originalDimensions = ref({ width: 0, height: 0 })

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

// Convert widget position to grid coordinates based on widget's center
const getGridPositionFromWidget = (widgetX: number, widgetY: number, gridElement: HTMLElement) => {
  const rect = gridElement.getBoundingClientRect()

  // Calculate widget's center point
  const widgetCenterX = widgetX + originalDimensions.value.width / 2
  const widgetCenterY = widgetY + originalDimensions.value.height / 2

  // Convert to relative position within grid
  const relativeX = widgetCenterX - rect.left - 8 // subtract container padding
  const relativeY = widgetCenterY - rect.top - 8

  // Calculate cell size including gaps
  const cellWidth = (rect.width - 8 * (props.columns - 1) - 16) / props.columns // 16 = 8px padding each side
  const cellHeight = (rect.height - 8 * (props.rows - 1) - 16) / props.rows

  // Calculate grid position based on center
  const gridX = Math.floor(relativeX / (cellWidth + 8))
  const gridY = Math.floor(relativeY / (cellHeight + 8))

  return {
    x: Math.max(0, Math.min(gridX, props.columns - props.widget.size.width)),
    y: Math.max(0, Math.min(gridY, props.rows - props.widget.size.height)),
  }
}

// Mouse event handlers
const handleMouseDown = (event: MouseEvent) => {
  event.preventDefault()

  // Capture the current dimensions before starting drag
  const element = event.currentTarget as HTMLElement
  const rect = element.getBoundingClientRect()

  originalDimensions.value = {
    width: rect.width,
    height: rect.height,
  }

  // Calculate where in the widget we clicked (relative to widget's top-left)
  clickOffset.value = {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top,
  }

  isDragging.value = true
  originalPosition.value = { ...props.widget.position }

  // Initial position: place widget so click point is under cursor
  dragOffset.value = {
    x: event.clientX - clickOffset.value.x,
    y: event.clientY - clickOffset.value.y,
  }

  // Add global event listeners
  document.addEventListener('mousemove', handleMouseMove)
  document.addEventListener('mouseup', handleMouseUp)

  // Prevent text selection while dragging
  document.body.style.userSelect = 'none'
}

const handleMouseMove = (event: MouseEvent) => {
  if (!isDragging.value) return

  // Simply update position: mouse position minus the click offset
  dragOffset.value = {
    x: event.clientX - clickOffset.value.x,
    y: event.clientY - clickOffset.value.y,
  }
}

const handleMouseUp = () => {
  if (!isDragging.value) return

  // Find the grid container element
  const gridElement = document.querySelector('.grid-wrapper') as HTMLElement
  if (!gridElement) {
    resetDrag()
    return
  }

  // Calculate new grid position based on widget's position, not mouse position
  const newGridPos = getGridPositionFromWidget(dragOffset.value.x, dragOffset.value.y, gridElement)

  // Check if the new position would cause overlap
  if (wouldOverlap(newGridPos.x, newGridPos.y)) {
    // Return to original position - no position update needed
    console.log('Overlap detected, returning to original position')
  } else {
    // Valid position - emit update event
    emit('updateWidget', {
      id: props.widget.id,
      position: newGridPos,
    })
  }

  resetDrag()
}

const resetDrag = () => {
  isDragging.value = false

  // Remove global event listeners
  document.removeEventListener('mousemove', handleMouseMove)
  document.removeEventListener('mouseup', handleMouseUp)

  // Re-enable text selection
  document.body.style.userSelect = ''
}

// Touch event handlers for mobile support
const handleTouchStart = (event: TouchEvent) => {
  event.preventDefault()
  const touch = event.touches[0]
  const element = event.currentTarget as HTMLElement
  const rect = element.getBoundingClientRect()

  // Capture dimensions
  originalDimensions.value = {
    width: rect.width,
    height: rect.height,
  }

  // Calculate click offset for touch
  clickOffset.value = {
    x: touch.clientX - rect.left,
    y: touch.clientY - rect.top,
  }

  isDragging.value = true
  originalPosition.value = { ...props.widget.position }

  // Initial position for touch
  dragOffset.value = {
    x: touch.clientX - clickOffset.value.x,
    y: touch.clientY - clickOffset.value.y,
  }

  document.addEventListener('touchmove', handleTouchMove, { passive: false })
  document.addEventListener('touchend', handleTouchEnd)
}

const handleTouchMove = (event: TouchEvent) => {
  if (!isDragging.value) return
  event.preventDefault()

  const touch = event.touches[0]

  // Simple calculation for touch move
  dragOffset.value = {
    x: touch.clientX - clickOffset.value.x,
    y: touch.clientY - clickOffset.value.y,
  }
}

const handleTouchEnd = () => {
  if (!isDragging.value) return

  const gridElement = document.querySelector('.grid-wrapper') as HTMLElement

  if (!gridElement) {
    resetTouchDrag()
    return
  }

  // Calculate new grid position based on widget position
  const newGridPos = getGridPositionFromWidget(dragOffset.value.x, dragOffset.value.y, gridElement)

  if (wouldOverlap(newGridPos.x, newGridPos.y)) {
    console.log('Overlap detected, returning to original position')
  } else {
    emit('updateWidget', {
      id: props.widget.id,
      position: newGridPos,
    })
  }

  resetTouchDrag()
}

const resetTouchDrag = () => {
  isDragging.value = false
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

      <div v-if="widget.type === 'text'" class="text-widget">
        <h3>Text Widget</h3>
        <p>This is a placeholder text widget</p>
      </div>
      <div v-else-if="widget.type === 'image'" class="image-widget">
        <h3>Image Widget</h3>
        <div class="image-placeholder">📷</div>
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
  /* Remove any transitions that could cause dampening */
}

.widget:hover {
  border-color: #4f46e5;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.2);
}

.widget-dragging {
  cursor: grabbing;
  border-color: #4f46e5;
  box-shadow: 0 8px 24px rgba(79, 70, 229, 0.3);
  /* No transition on dragging state for immediate response */
}

.widget-content {
  height: 100%;
  padding: 12px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  position: relative;
}

.drag-handle {
  position: absolute;
  top: 8px;
  right: 8px;
  color: #666;
  opacity: 0;
  transition: opacity 0.2s ease;
  pointer-events: none;
}

.widget:hover .drag-handle {
  opacity: 1;
}

.text-widget h3,
.image-widget h3 {
  margin: 0 0 8px 0;
  color: #f2faff;
  font-size: 1rem;
}

.text-widget p {
  margin: 0;
  color: #97968a;
  font-size: 0.875rem;
}

.image-placeholder {
  font-size: 2rem;
  opacity: 0.6;
}
</style>

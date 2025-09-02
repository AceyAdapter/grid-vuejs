<script setup lang="ts">
interface Widget {
  id: string
  type: 'text' | 'image'
  position: { x: number; y: number }
  size: { width: number; height: number }
  content: unknown
}

interface Props {
  widget: Widget
}

defineProps<Props>()
</script>

<template>
  <div
    class="widget"
    :style="{
      gridColumn: `${widget.position.x + 1} / span ${widget.size.width}`,
      gridRow: `${widget.position.y + 1} / span ${widget.size.height}`,
    }"
  >
    <div class="widget-content">
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
  transition: all 0.2s ease;
  cursor: pointer;
}

.widget:hover {
  border-color: #4f46e5;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.2);
}

.widget-content {
  height: 100%;
  padding: 12px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
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

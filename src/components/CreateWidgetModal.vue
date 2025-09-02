<script setup lang="ts">
import { ref } from 'vue'

interface Props {
  isOpen: boolean
  columns: number
}

defineProps<Props>()

// Events this component can emit
const emit = defineEmits<{
  close: []
  createWidget: [{ type: 'text' | 'image'; width: number; height: number }]
}>()

const selectedType = ref<'text' | 'image'>('text')
const width = ref(2)
const height = ref(2)

const handleBackdropClick = (event: MouseEvent) => {
  // Only close if clicking the backdrop, not the modal content
  if (event.target === event.currentTarget) {
    emit('close')
  }
}

const handleCreate = () => {
  emit('createWidget', {
    type: selectedType.value,
    width: width.value,
    height: height.value,
  })
  emit('close')
}

const handleClose = () => {
  emit('close')
}

// Handle escape key
const handleKeydown = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    emit('close')
  }
}
</script>

<template>
  <!-- v-if is Vue's conditional rendering (like {condition && <div>} in React) -->
  <Teleport to="body">
    <div
      v-if="isOpen"
      class="modal-backdrop"
      @click="handleBackdropClick"
      @keydown="handleKeydown"
      tabindex="-1"
    >
      <div class="modal-content">
        <div class="modal-header">
          <h2 class="modal-title">Create Widget</h2>
          <button @click="handleClose" class="close-button">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
              <path
                d="M18 6L6 18M6 6L18 18"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
            </svg>
          </button>
        </div>

        <div class="modal-body">
          <!-- Widget Type Selection -->
          <div class="form-section">
            <label class="form-label">Widget Type</label>
            <div class="widget-type-grid">
              <button
                @click="selectedType = 'text'"
                :class="['widget-type-card', { active: selectedType === 'text' }]"
              >
                <div class="widget-icon">📝</div>
                <span>Text Widget</span>
              </button>
              <button
                @click="selectedType = 'image'"
                :class="['widget-type-card', { active: selectedType === 'image' }]"
              >
                <div class="widget-icon">🖼️</div>
                <span>Image Widget</span>
              </button>
            </div>
          </div>

          <!-- Size Configuration -->
          <div class="form-section">
            <label class="form-label">Size</label>
            <div class="size-controls">
              <div class="size-control">
                <label for="width">Width: {{ width }} columns</label>
                <input
                  id="width"
                  v-model.number="width"
                  type="range"
                  min="1"
                  :max="columns"
                  class="slider"
                />
              </div>
              <div class="size-control">
                <label for="height">Height: {{ height }} rows</label>
                <input
                  id="height"
                  v-model.number="height"
                  type="range"
                  min="1"
                  max="6"
                  class="slider"
                />
              </div>
            </div>
          </div>

          <!-- Preview -->
          <div class="form-section">
            <label class="form-label">Preview</label>
            <div class="size-preview">
              <div class="preview-grid" :style="{ gridTemplateColumns: `repeat(${columns}, 1fr)` }">
                <div
                  v-for="i in columns * 3"
                  :key="i"
                  :class="[
                    'preview-cell',
                    {
                      'preview-selected': i <= width && Math.ceil(i / columns) <= height,
                    },
                  ]"
                ></div>
              </div>
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button @click="handleClose" class="button-secondary">Cancel</button>
          <button @click="handleCreate" class="button-primary">Create Widget</button>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.2s ease;
}

.modal-content {
  background-color: #2a2a2a;
  border-radius: 12px;
  border: 1px solid #404040;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideIn 0.2s ease;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px 16px;
  border-bottom: 1px solid #404040;
}

.modal-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #f2faff;
  margin: 0;
}

.close-button {
  background: none;
  border: none;
  color: #97968a;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: all 0.2s ease;
}

.close-button:hover {
  background-color: #404040;
  color: #f2faff;
}

.modal-body {
  padding: 20px 24px;
}

.form-section {
  margin-bottom: 24px;
}

.form-label {
  display: block;
  font-weight: 500;
  color: #f2faff;
  margin-bottom: 8px;
  font-size: 0.875rem;
}

.widget-type-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.widget-type-card {
  background-color: #404040;
  border: 2px solid #555;
  border-radius: 8px;
  padding: 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #97968a;
}

.widget-type-card:hover {
  border-color: #666;
  background-color: #4a4a4a;
}

.widget-type-card.active {
  border-color: #4f46e5;
  background-color: #4f46e5;
  color: white;
}

.widget-icon {
  font-size: 2rem;
}

.size-controls {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.size-control {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.size-control label {
  font-size: 0.875rem;
  color: #97968a;
}

.slider {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: #404040;
  outline: none;
  -webkit-appearance: none;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
  border: none;
}

.size-preview {
  background-color: #1a1a1a;
  border-radius: 8px;
  padding: 16px;
  border: 1px solid #404040;
}

.preview-grid {
  display: grid;
  gap: 4px;
  max-width: 200px;
}

.preview-cell {
  aspect-ratio: 1;
  background-color: #404040;
  border-radius: 3px;
  transition: all 0.2s ease;
}

.preview-selected {
  background-color: #4f46e5;
}

.modal-footer {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding: 16px 24px 20px;
  border-top: 1px solid #404040;
}

.button-secondary {
  background-color: #404040;
  color: #97968a;
  border: 1px solid #555;
  border-radius: 6px;
  padding: 8px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.button-secondary:hover {
  background-color: #4a4a4a;
  border-color: #666;
}

.button-primary {
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 8px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.button-primary:hover {
  background-color: #4338ca;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}
</style>

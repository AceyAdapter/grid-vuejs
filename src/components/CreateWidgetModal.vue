<script setup lang="ts">
import { ref, computed } from 'vue'

interface Props {
  isOpen: boolean
  columns: number
}

type ContentType = {
  text?: string
  textColor?: string
  backgroundColor?: string
  url?: string
  localFile?: string
}

defineProps<Props>()

// Events this component can emit
const emit = defineEmits<{
  close: []
  createWidget: [
    {
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
    },
  ]
}>()

const selectedType = ref<'text' | 'image'>('text')
const width = ref(2)
const height = ref(2)

// Text widget properties
const textContent = ref('Sample Text')
const textColor = ref('#f2faff')
const backgroundColor = ref('#4f46e5')

// Image widget properties
const imageUrl = ref('')
const uploadedImage = ref<string | null>(null)
const useUpload = ref(true) // Toggle between upload and URL

// File upload handling
const handleFileUpload = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]

  if (!file) return

  // Check file size (10MB limit)
  if (file.size > 10 * 1024 * 1024) {
    alert('File size exceeds 10MB limit')
    return
  }

  // Check file type
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp', 'image/svg+xml']
  if (!allowedTypes.includes(file.type)) {
    alert('Unsupported file format. Please use JPG, PNG, GIF, WebP, or SVG.')
    return
  }

  // Convert to base64 for localStorage storage
  const reader = new FileReader()
  reader.onload = (e) => {
    uploadedImage.value = e.target?.result as string
  }
  reader.readAsDataURL(file)
}

// Computed property for image source (Vue's equivalent to useMemo)
const imageSource = computed(() => {
  if (useUpload.value) {
    return uploadedImage.value || ''
  } else {
    return imageUrl.value
  }
})

// Computed styles for text preview
const textPreviewStyle = computed(() => ({
  color: textColor.value,
  backgroundColor: backgroundColor.value,
  padding: '16px',
  borderRadius: '8px',
  minHeight: '80px',
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center',
  fontSize: '1rem',
  fontWeight: 'normal',
}))

const handleBackdropClick = (event: MouseEvent) => {
  if (event.target === event.currentTarget) {
    emit('close')
  }
}

const handleCreate = () => {
  if (selectedType.value === 'text') {
    emit('createWidget', {
      type: selectedType.value,
      width: width.value,
      height: height.value,
      content: {
        text: textContent.value,
        textColor: textColor.value,
        backgroundColor: backgroundColor.value,
      },
    })
  } else if (selectedType.value === 'image') {
    const content: ContentType = {}

    if (useUpload.value && uploadedImage.value) {
      content.localFile = uploadedImage.value
    } else if (!useUpload.value && imageUrl.value) {
      content.url = imageUrl.value
    } else {
      alert('Please provide an image')
      return
    }

    emit('createWidget', {
      type: selectedType.value,
      width: width.value,
      height: height.value,
      content,
    })
  }

  emit('close')
}

const handleClose = () => {
  emit('close')
}

// Reset form when type changes
const handleTypeChange = (type: 'text' | 'image') => {
  selectedType.value = type
  // Reset form values to defaults
  if (type === 'text') {
    textContent.value = 'Sample Text'
    textColor.value = '#f2faff'
    backgroundColor.value = '#4f46e5'
  } else {
    imageUrl.value = ''
    uploadedImage.value = null
    useUpload.value = true
  }
}

// Handle escape key
const handleKeydown = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    emit('close')
  }
}
</script>

<template>
  <Teleport to="body">
    <div
      v-if="isOpen"
      class="modal-backdrop"
      @mousedown="handleBackdropClick"
      @keydown="handleKeydown"
      tabindex="-1"
    >
      <div class="modal-container">
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
                  @click="handleTypeChange('text')"
                  :class="['widget-type-card', { active: selectedType === 'text' }]"
                >
                  <div class="widget-icon">📝</div>
                  <span>Text Widget</span>
                </button>
                <button
                  @click="handleTypeChange('image')"
                  :class="['widget-type-card', { active: selectedType === 'image' }]"
                >
                  <div class="widget-icon">🖼️</div>
                  <span>Image Widget</span>
                </button>
              </div>
            </div>

            <!-- Text Widget Configuration -->
            <div v-if="selectedType === 'text'" class="form-section">
              <label class="form-label">Text Content</label>
              <input
                class="text-input"
                type="text"
                v-model="textContent"
                placeholder="Enter your text..."
              />

              <div class="color-controls">
                <div class="color-control">
                  <label for="text-color">Text Color</label>
                  <div class="color-input-group">
                    <input id="text-color" type="color" v-model="textColor" class="color-picker" />
                    <span class="color-value">{{ textColor }}</span>
                  </div>
                </div>

                <div class="color-control">
                  <label for="bg-color">Background Color</label>
                  <div class="color-input-group">
                    <input
                      id="bg-color"
                      type="color"
                      v-model="backgroundColor"
                      class="color-picker"
                    />
                    <span class="color-value">{{ backgroundColor }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Image Widget Configuration -->
            <div v-if="selectedType === 'image'" class="form-section">
              <label class="form-label">Image Source</label>

              <div class="image-source-toggle">
                <button @click="useUpload = true" :class="['toggle-button', { active: useUpload }]">
                  Upload File
                </button>
                <button
                  @click="useUpload = false"
                  :class="['toggle-button', { active: !useUpload }]"
                >
                  URL
                </button>
              </div>

              <div v-if="useUpload" class="upload-section">
                <input
                  type="file"
                  accept="image/*"
                  @change="handleFileUpload"
                  class="file-input"
                  id="file-upload"
                />
                <label for="file-upload" class="file-upload-label">
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                    <path
                      d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"
                      stroke="currentColor"
                      stroke-width="2"
                    />
                    <polyline points="7,10 12,15 17,10" stroke="currentColor" stroke-width="2" />
                    <line x1="12" y1="15" x2="12" y2="3" stroke="currentColor" stroke-width="2" />
                  </svg>
                  <span class="upload-text">{{
                    uploadedImage ? 'Change File' : 'Choose File'
                  }}</span>
                </label>
                <p class="file-info">Max 10MB • JPG, PNG, GIF, WebP, SVG</p>
              </div>

              <div v-else class="url-section">
                <input
                  class="text-input"
                  type="url"
                  v-model="imageUrl"
                  placeholder="https://example.com/image.jpg"
                />
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

            <!-- Widget Preview -->
            <div class="form-section">
              <label class="form-label">Preview</label>
              <div class="widget-preview">
                <!-- Text Widget Preview -->
                <div
                  v-if="selectedType === 'text'"
                  class="preview-widget text-preview"
                  :style="textPreviewStyle"
                >
                  {{ textContent || 'Sample Text' }}
                </div>

                <!-- Image Widget Preview -->
                <div v-else-if="selectedType === 'image'" class="preview-widget image-preview">
                  <img
                    v-if="imageSource"
                    :src="imageSource"
                    alt="Preview"
                    class="preview-image"
                    @error="() => {}"
                  />
                  <div v-else class="image-placeholder">
                    <svg width="48" height="48" viewBox="0 0 24 24" fill="none">
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
                    <p>No image selected</p>
                  </div>
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
  /* Ensure proper scrolling on mobile */
  overflow-y: auto;
  padding: 16px;
}

.modal-container {
  /* Container for centering and constraining modal size */
  width: 100%;
  max-width: 500px;
  max-height: calc(100vh - 32px);
  display: flex;
  flex-direction: column;
  /* Allow modal to shrink on very small screens */
  min-height: 0;
}

.modal-content {
  background-color: #2a2a2a;
  border-radius: 12px;
  border: 1px solid #404040;
  display: flex;
  flex-direction: column;
  /* Ensure modal can scroll internally */
  max-height: 100%;
  overflow: hidden;
  animation: slideIn 0.2s ease;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid #404040;
  /* Prevent header from shrinking */
  flex-shrink: 0;
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
  padding: 8px;
  border-radius: 4px;
  transition: all 0.2s ease;
  /* Ensure touch target is adequate */
  min-width: 40px;
  min-height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-button:hover {
  background-color: #404040;
  color: #f2faff;
}

.modal-body {
  /* Allow body to scroll while keeping header/footer fixed */
  overflow-y: auto;
  padding: 20px;
  flex: 1;
  /* Minimum height to prevent content from being too cramped */
  min-height: 0;
}

.form-section {
  margin-bottom: 24px;
}

.form-section:last-child {
  margin-bottom: 0;
}

.form-label {
  display: block;
  font-weight: 500;
  color: #f2faff;
  margin-bottom: 12px;
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
  padding: 16px 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #97968a;
  /* Ensure adequate touch targets */
  min-height: 80px;
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
  font-size: 1.5rem;
}

.widget-type-card span {
  font-size: 0.875rem;
  text-align: center;
}

.text-input {
  width: 100%;
  padding: 12px;
  border: 1px solid #555;
  border-radius: 6px;
  background: #404040;
  color: #f2faff;
  outline: none;
  transition: border-color 0.2s ease;
  margin-bottom: 16px;
  /* Ensure input doesn't overflow on mobile */
  box-sizing: border-box;
  font-size: 16px; /* Prevents zoom on iOS */
}

.text-input:focus {
  border-color: #4f46e5;
}

.color-controls {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.color-control {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.color-control label {
  font-size: 0.875rem;
  color: #97968a;
}

.color-input-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.color-picker {
  width: 60px;
  height: 40px;
  border: 1px solid #555;
  border-radius: 6px;
  background: #404040;
  cursor: pointer;
  padding: 2px;
  /* Remove default appearance for better mobile experience */
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

.color-picker::-webkit-color-swatch-wrapper {
  padding: 0;
}

.color-picker::-webkit-color-swatch {
  border: none;
  border-radius: 4px;
}

.color-value {
  font-size: 0.75rem;
  color: #666;
  font-family: monospace;
  flex: 1;
}

.image-source-toggle {
  display: flex;
  background-color: #404040;
  border-radius: 6px;
  border: 1px solid #555;
  margin-bottom: 16px;
  overflow: hidden;
}

.toggle-button {
  flex: 1;
  padding: 12px 8px;
  background: none;
  border: none;
  color: #97968a;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.875rem;
}

.toggle-button.active {
  background-color: #4f46e5;
  color: white;
}

.upload-section {
  text-align: center;
}

.file-input {
  display: none;
}

.file-upload-label {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 16px 20px;
  background-color: #404040;
  border: 2px dashed #666;
  border-radius: 8px;
  color: #97968a;
  cursor: pointer;
  transition: all 0.2s ease;
  width: 100%;
  box-sizing: border-box;
  min-height: 60px;
}

.file-upload-label:hover {
  border-color: #4f46e5;
  background-color: #4a4a4a;
}

.upload-text {
  font-size: 0.875rem;
}

.file-info {
  margin: 8px 0 0;
  font-size: 0.75rem;
  color: #666;
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
  appearance: none;
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

.widget-preview {
  background-color: #1a1a1a;
  border-radius: 8px;
  padding: 16px;
  border: 1px solid #404040;
}

.preview-widget {
  border-radius: 8px;
  min-height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.text-preview {
  word-wrap: break-word;
  /* Ensure text doesn't overflow preview */
  overflow-wrap: break-word;
  hyphens: auto;
}

.image-preview {
  background-color: #404040;
  position: relative;
  overflow: hidden;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 4px;
  max-height: 200px;
}

.image-placeholder {
  color: #666;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.image-placeholder p {
  margin: 0;
  font-size: 0.875rem;
}

.modal-footer {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding: 16px 20px;
  border-top: 1px solid #404040;
  /* Prevent footer from shrinking */
  flex-shrink: 0;
}

.button-secondary,
.button-primary {
  padding: 12px 20px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.875rem;
  /* Ensure adequate touch targets */
  min-height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.button-secondary {
  background-color: #404040;
  color: #97968a;
  border: 1px solid #555;
}

.button-secondary:hover {
  background-color: #4a4a4a;
  border-color: #666;
}

.button-primary {
  background-color: #4f46e5;
  color: white;
  border: none;
}

.button-primary:hover {
  background-color: #4338ca;
}

/* Mobile-specific improvements */
@media (max-width: 480px) {
  .modal-backdrop {
    padding: 8px;
    align-items: flex-start;
    padding-top: 20px;
  }

  .modal-container {
    max-height: calc(100vh - 40px);
  }

  .modal-header {
    padding: 12px 16px;
  }

  .modal-body {
    padding: 16px;
  }

  .modal-footer {
    padding: 12px 16px;
    flex-direction: column;
  }

  .button-secondary,
  .button-primary {
    width: 100%;
  }

  .color-controls {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .widget-type-grid {
    gap: 8px;
  }

  .widget-type-card {
    padding: 12px 8px;
    min-height: 70px;
  }

  .widget-icon {
    font-size: 1.25rem;
  }

  .form-section {
    margin-bottom: 20px;
  }
}

/* Small mobile devices */
@media (max-width: 360px) {
  .modal-backdrop {
    padding: 4px;
    padding-top: 10px;
  }

  .color-input-group {
    flex-direction: column;
    align-items: stretch;
  }

  .color-picker {
    width: 100%;
  }
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

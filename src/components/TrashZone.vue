<script setup lang="ts">
interface Props {
  isVisible: boolean
  isActive: boolean
}

defineProps<Props>()

const emit = defineEmits<{
  drop: []
}>()

const handleDrop = () => {
  emit('drop')
}
</script>

<template>
  <div
    v-if="isVisible"
    class="trash-zone"
    :class="{ 'trash-active': isActive }"
    @mouseup="handleDrop"
    @touchend="handleDrop"
  >
    <div class="trash-icon">
      <svg width="32" height="32" viewBox="0 0 24 24" fill="none">
        <path
          d="M3 6h18m-2 0v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
        <line x1="10" y1="11" x2="10" y2="17" stroke="currentColor" stroke-width="2" />
        <line x1="14" y1="11" x2="14" y2="17" stroke="currentColor" stroke-width="2" />
      </svg>
    </div>
    <span class="trash-text">Drop to Delete</span>
  </div>
</template>

<style scoped>
.trash-zone {
  position: fixed;
  bottom: 24px;
  left: 24px;
  width: 200px;
  height: 80px;
  background-color: rgba(239, 68, 68, 0.9);
  border: 2px dashed #dc2626;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  color: white;
  font-size: 0.875rem;
  font-weight: 500;
  z-index: 999;
  transition: all 0.2s ease;
  backdrop-filter: blur(4px);
  opacity: 0.7;
}

.trash-zone.trash-active {
  opacity: 1;
  transform: scale(1.1);
  box-shadow: 0 8px 24px rgba(239, 68, 68, 0.4);
  background-color: rgba(220, 38, 38, 0.95);
}

.trash-icon {
  transition: transform 0.2s ease;
}

.trash-active .trash-icon {
  transform: scale(1.2);
  animation: bounce 0.5s ease-in-out infinite alternate;
}

.trash-text {
  font-size: 0.75rem;
  text-align: center;
}

@keyframes bounce {
  from {
    transform: scale(1.2) translateY(0);
  }
  to {
    transform: scale(1.2) translateY(-4px);
  }
}

/* Mobile adjustments */
@media (max-width: 767px) {
  .trash-zone {
    width: 150px;
    height: 60px;
    bottom: 20px;
    left: 20px;
  }

  .trash-icon svg {
    width: 24px;
    height: 24px;
  }

  .trash-text {
    font-size: 0.625rem;
  }
}
</style>

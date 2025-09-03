<script setup lang="ts">
const handleStarClick = () => {
  window.open('https://github.com/AceyAdapter/grid-vuejs', '_blank')
}
</script>

<template>
  <button @click="handleStarClick" class="github-star-button" title="Star on GitHub">
    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path
        d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"
        stroke="currentColor"
        stroke-width="2"
        stroke-linejoin="round"
      />
    </svg>
    <span>Star on GitHub</span>
  </button>
</template>

<style scoped>
.github-star-button {
  margin-top: 12px;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background-color: #2f2f2f;
  color: #97968a;
  border: 1px solid #555;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.875rem;
  margin-left: 12px;
}

.github-star-button:hover {
  background-color: #4a4a4a;
  color: #f2faff;
  border-color: #666;
  transform: translateY(-1px);
}

.github-star-button:active {
  transform: translateY(0);
}

.github-star-button span {
  font-weight: 500;
}
</style>

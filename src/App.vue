<script setup lang="ts">
import GridContainer from './components/GridContainer.vue'

const gridData = [1, 2, 3, 4, 5, 6, 7]
</script>

<template>
  <div class="grid">
    <GridContainer v-for="item in gridData" :key="item" />
  </div>
</template>

<style scoped>
.grid {
  display: flex;
  width: auto;
  gap: 10px;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}
</style>

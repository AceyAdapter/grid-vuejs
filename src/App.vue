<script setup lang="ts">
import GridContainer from './components/GridContainer.vue'
import GitHubStarButton from './components/GitHubStarButton.vue'
</script>

<template>
  <div class="app">
    <header class="app-header">
      <div class="header-title">
        <h1 class="primary">The Grid</h1>
      </div>

      <p class="secondary subtitle">Customizable Widget Dashboard made with Vue 3</p>
      <GitHubStarButton />
    </header>

    <main class="app-main">
      <GridContainer />
    </main>

    <footer>
      <p>
        Made with 💜 by
        <a
          href="https://www.linkedin.com/in/nicholas-kreitz-3a13531a0/"
          target="_blank"
          class="profile-link"
          >Nicholas Kreitz</a
        >
      </p>
    </footer>
  </div>
</template>

<style scoped>
.header-title {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px;
}

.app-header {
  text-align: center;
  margin-bottom: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.app-header h1 {
  font-size: 2.5rem;
  margin: 0 0 0.5rem 0;
  font-weight: bold;
}

.app-header p {
  font-size: 1.1rem;
  margin: 0;
}

.app-main {
  flex: 1;
  width: 100%;
  max-width: 1200px;
}

.profile-link {
  color: gray;
}

@media (max-width: 767px) {
  .app {
    padding: 12px;
  }

  .app-header h1 {
    font-size: 2rem;
  }

  .app-header p {
    font-size: 1rem;
  }

  .header-title {
    flex-direction: column;
    gap: 8px;
  }

  .subtitle {
    width: 75%;
  }
}
</style>

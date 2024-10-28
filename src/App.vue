<template>
  <div class="app">
    <PacmanGame />
  </div>
</template>

<script>
import PacmanGame from './components/PacmanGame.vue'

export default {
  name: 'App',
  components: {
    PacmanGame
  }
}
</script>

<style>
body {
  margin: 0;
  background: #000;
}
.app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
</style>
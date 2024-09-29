<template>
  <div id="app">
    <indexPage />
  </div>
</template>

<script>
import indexPage from './components/indexPage.vue';

export default {
  name: 'App',
  components: {
    indexPage
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>

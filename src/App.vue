<template>
  <div id="app">
    <pt></pt>
  </div>
</template>

<script>
import pt from './components/pt.vue'

export default {
  name: 'app',
  components: {
    pt
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>

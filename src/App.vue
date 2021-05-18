<template>
  <div id="app">
    <VirtualGallery />
  </div>
</template>

<script>
import VirtualGallery from './components/VirtualGallery.vue'

export default {
  name: 'App',
  components: {
   VirtualGallery 
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
  margin-top: 60px;
}
</style>

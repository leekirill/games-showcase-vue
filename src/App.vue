<template>
  <div>
    <app-home/>
  </div>
</template>

<script>
import AppHome from './components/AppHome.vue'

export default {
  name: 'App',
  components: {
    AppHome
  }
}
</script>

<style>
body {
    background-color: #171717;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
  margin: auto;
  max-width: 1200px;
}
</style>

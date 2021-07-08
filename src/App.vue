<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <Viewer></Viewer>
  <a target="_blank" href="https://github.com/ToshihitoKon/panepon-maker">github</a>
</template>

<script>
import Viewer from './pages/Viewer.vue'

export default {
  name: 'App',
  components: {
    Viewer,
  }
}
</script>

<style>
#app{
  text-align: center;
}
</style>

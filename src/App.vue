<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/Lunala_logo.jpg" style="width:300px; height:300px;">
    <Main/>
  </div>
</template>

<script>
import Main from './components/Main.vue'

export default {
  name: 'App',
  components: {
    Main
  },
  created() {
    document.title = "L.U.N.A.L.A";
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

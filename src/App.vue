<template>
  <img alt="Vue logo" src="./assets/logo.webp" class="logo"> 
  <TestCamera />
</template>

<script>
import TestCamera from './components/TestCamera.vue';

export default {
  name: 'App',
  components: {
    TestCamera
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

.logo {
  height: 70px;
  width: 120px;
}
</style>

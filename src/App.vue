<template>
  <div id="app">
    <img alt="Random Logo" src="https://logo.clearbit.com/google.com">
    <div>Not what you think?</div>
    <SearchField msg="Google rank checker"/>
  </div>
</template>

<script>
import SearchField from './components/Search.vue'

export default {
  name: 'App',
  components: {
    SearchField
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

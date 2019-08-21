<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
* {
  box-sizing: border-box;
  margin:0;
  padding: 0;
}

body {
  font-family: sans-serif;
  font-size: 14px;
  color: #333;
  -webkit-font-smoothing: antialiased;
}
</style>

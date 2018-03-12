<template>
  <div id="app">
    <!--<img src="./assets/logo.png">-->
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Montserrat|Open+Sans');

body {
  margin: 0;
}

h1,h2,h3,h4,h5 {
  font-family: 'Montserrat', sans-serif;
}

p {
  font-family: 'Open Sans', sans-serif;
}
</style>

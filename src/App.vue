<template>
  <div id="app">
    <div class="logo">
      <img src="./assets/logo.png">
    </div>
    <datepicker></datepicker>
  </div>
</template>

<script>
import Datepicker from './components/datepicker'

export default {
  name: 'app',
  components: {
    Datepicker
  }
}
</script>

<style>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
  }
  .logo {
    padding-top: 50px;
    text-align: center;
  }

</style>

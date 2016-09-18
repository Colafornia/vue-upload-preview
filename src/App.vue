<template>
  <div id="app">
    <file-upload></file-upload>
  </div>
</template>

<script>
import fileUpload from './components/fileUpload'
import css from './assets/file-upload.css'
import bootstrap from 'bootstrap/dist/css/bootstrap.css'

export default {
  components: {
    fileUpload
  }
}
</script>

<style>
body {
  width: 100%;
  height: 100%;
}

#app {
  color: #2c3e50;
  font-family: Source Sans Pro, Helvetica, sans-serif;
}

#app a {
  color: #42b983;
  text-decoration: none;
}
</style>

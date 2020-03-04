<template>
  <div id="app">
    <QrReader result="" error="" />
  </div>
</template>

<script>
import QrReader from './components/QrReader'

export default {
  name: 'app',
  components: {
    QrReader
  },
  mounted: function(){
    if (/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
      // When ready, auto-scroll 1px to hide URL bar
      window.addEventListener("load", function() {
        // Set a timeout...
        setTimeout(function() {
          // Hide the address bar!
          window.scrollTo(0, 1);
        }, 0);
      });
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.header {
  background-color: black;
  padding: 10px 5px;
  color: white;
  text-transform: uppercase;
  font-weight: bold;
}

.modal-backdrop {
  display: none;
}
</style>

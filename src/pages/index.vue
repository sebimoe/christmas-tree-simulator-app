<template>
  <MainTreePanel />
</template>

<script>
export default {
  mounted() {
    setTimeout(() => document.body.classList.add("dom-ready"), 1);
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600&display=swap');

html, body {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
}

html, body, input, textarea, button, select {
  font-family: 'Open Sans', sans-serif;
}

body {
  &.dom-ready {
    transition: color 0.4s, background-color 0.4s;
  }

  &.dark-theme {
    background: #050505;
    color: #eee;

    button, input, select {
      opacity: 0.8;
    }
  }
}

body:not(.dark-theme) {
  .dark-theme-only {
    display: none !important;
  }
}
body.dark-theme {
  .light-theme-only {
    display: none !important;
  }
}

</style>

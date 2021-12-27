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

html, body {
  margin: 0;
  padding: 0;
}

html, body, input, textarea, button, select {
  font-family: sans-serif;
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

</style>

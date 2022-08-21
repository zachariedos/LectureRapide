<template>
  <readPage msg="Page de lecture" />
</template>

<script>
import readPage from "./components/readPage.vue";

export default {
  name: "App",
  components: {
    readPage,
  },
};
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

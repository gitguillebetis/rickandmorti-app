<template>
  <div id="app">
    <CharacterList />
  </div>
</template>

<script>
import CharacterList from './components/CharacterList.vue';

export default {
  components: {
    CharacterList
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 60px;
}
</style>

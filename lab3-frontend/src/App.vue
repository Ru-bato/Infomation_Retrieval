<template>
  <div id="app">
    <ImageSearch />
  </div>
</template>

<script>
import ImageSearch from './components/ImageSearch.vue';

export default {
  name: 'App',
  components: {
    ImageSearch,
  },
};
</script>

<style>
/* 全局样式 */
body {
  background: white;
}
</style>

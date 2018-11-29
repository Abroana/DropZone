<template>
  <div id="app">
    <drop-zone/>
  </div>
</template>

<script>
export default {
  name: 'app',
  components: {
    DropZone: () => import('./components/DropZone.vue'),
  },
}
</script>

<style lang="postcss">
@import url('./assets/css/common.css');

#app {
  text-align: center;
  padding-top: 30px;
}
</style>


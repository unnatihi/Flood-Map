<template>
  <div id="app">
    <OpenLayersMap />
  </div>
</template>

<script setup lang="ts">
import OpenLayersMap from './components/OpenLayersMap.vue';
</script>

<style>
html, body, #app {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
</style>

<template>
  <div id="app">
    <div style="display: inline-flex;flex-wrap: nowrap">
      <map-container ></map-container>
    </div>
  </div>
</template>

<script>
import MapContainer from "@/components/MapContainer";

export default {
  name: 'App',
  components: {
    MapContainer
  }
}
</script>

<style>
html, body {
  padding: 0;
  margin: 0;
}
</style>

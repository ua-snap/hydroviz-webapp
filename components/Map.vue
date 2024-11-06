<script setup lang="ts">
const { $L } = useNuxtApp()

onMounted(() => {
  var map = $L.map('map').setView([37.8, -96], 4)

  $L.tileLayer(
    'https://basemap.nationalmap.gov/arcgis/rest/services/USGSTopo/MapServer/tile/{z}/{y}/{x}',
    {
      maxZoom: 19,
      attribution: 'Map data © USGS',
    }
  ).addTo(map)

  $L.tileLayer
    .wms('https://gs.earthmaps.io/geoserver/wms', {
      transparent: true,
      format: 'image/png',
      layers: 'hydrology:seg',
      zIndex: 10,
    })
    .addTo(map)

  // Print the current Leaflet zoom level out to the console
  map.on('zoomend', function () {
    console.log('Current Zoom: ' + map.getZoom())
  })
})
</script>

<template>
  <div>
    <div id="map" style="height: 500px"></div>
  </div>
</template>

<style scoped></style>

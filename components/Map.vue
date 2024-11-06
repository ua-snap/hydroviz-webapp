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

  var wmsLayer = $L.tileLayer
    .wms('https://gs.earthmaps.io/geoserver/wms', {
      transparent: true,
      format: 'image/png',
      layers: 'hydrology:seg',
      zIndex: 10,
    })
    .addTo(map)

  map.on('zoomend', function () {
    if (map.getZoom() > 8) {
      var bounds = map.getBounds()
      var minLon = bounds.getWest()
      var maxLon = bounds.getEast()
      var minLat = bounds.getSouth()
      var maxLat = bounds.getNorth()
      getGeoJSON(minLon, maxLon, maxLat, minLat)
    } else {
      map.addLayer(wmsLayer)
    }
  })

  const getGeoJSON = async (
    minLon: number,
    maxLon: number,
    maxLat: number,
    minLat: number
  ) => {
    fetch(
      'https://gs.earthmaps.io/geoserver/hydrology/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=hydrology%3Aseg&outputFormat=application%2Fjson&srsName=EPSG:4326&cql_filter=INTERSECTS(the_geom,ENVELOPE(' +
        minLon +
        ',' +
        maxLon +
        ',' +
        maxLat +
        ',' +
        minLat +
        '))'
    )
      .then(response => response.json())
      .then(data => {
        $L.geoJSON(data)
          .addTo(map)
          .on('click', function (e) {
            console.log(e.sourceTarget.feature.properties.seg_id_nat)
          })
        map.removeLayer(wmsLayer)
      })
  }

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

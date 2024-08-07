<template>
  <l-map ref="map" v-model:zoom="zoom" :center="[47.41322, -1.219482]">
    <l-tile-layer
      url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      layer-type="base"
      name="OpenStreetMap"
    ></l-tile-layer>

    <l-my-marker
      :lat-lng="coordinates"
      :draggable="true"
      @ready="markerReady"
    ></l-my-marker>
  </l-map>
</template>
<script setup lang="ts">
import type L from "leaflet";
import type { LatLngTuple } from "leaflet";
import { ref } from "vue";

import { LMap, LMyMarker, LTileLayer } from "@src/components";

const zoom = ref(2);
const coordinates = ref<LatLngTuple>([50, 50]);
function markerReady(event) {
  console.log("marker ready", event);
}
// export default {
//   components: {
//     LMap,
//     LTileLayer,
//     // LMarker,
//     LMyMarker,
//   },
//   data() {
//     return {
//       zoom: 2,
//       coordinates: [50, 50] as L.LatLngExpression,
//     };
//   },
// };
</script>

<style></style>

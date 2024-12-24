<template>
  <div id="map" class="map-container" :class="{ 'map-expanded': isSidebarCollapsed }"></div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'
import { mapOptions } from '../assets/data'
import { onMounted, onUnmounted } from 'vue'

export default {
  name: 'MapView',
  props: {
    isSidebarCollapsed: {
      type: Boolean,
      default: false
    }
  },
  setup() {
    let map = null

    onMounted(() => {
      mapboxgl.accessToken = 'pk.eyJ1Ijoia3BmdWkiLCJhIjoiY2p6MWcxMXl4MDFlbTNsbDc1bnp6N3FjYSJ9.66qFOXwI661MOPOf7x96yA'
      map = new mapboxgl.Map({
        ...mapOptions,
        container: 'map'
      })

      map.on('load', () => {
        console.debug('map loaded')
      })
    })

    onUnmounted(() => {
      if (map) {
        map.remove()
      }
    })

    return {}
  }
}
</script>

<style>
.map-container {
  width: 100vw;
  left: 0;
  top: 0;
  position: absolute;
  height: 100vh;
  overflow: hidden;
  transition: padding-left 0.3s ease;
  padding-left: var(--sidebar-width);
}

.map-expanded {
  padding-left: 0;
}
</style> 
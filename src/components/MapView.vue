<template>
  <div class="map-wrapper">
    <div id="map" class="map-container"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'
import { mapOptions } from '../assets/data'
import { onMounted, onUnmounted, watch } from 'vue'

export default {
  name: 'MapView',
  props: {
    isSidebarCollapsed: {
      type: Boolean,
      default: false
    },
    selectedLocation: {
      type: Object,
      default: () => null
    }
  },
  setup(props) {
    let map = null
    let currentCenter = null
    let mousePosition = null

    onMounted(() => {
      mapboxgl.accessToken = 'pk.eyJ1Ijoia3BmdWkiLCJhIjoiY2p6MWcxMXl4MDFlbTNsbDc1bnp6N3FjYSJ9.66qFOXwI661MOPOf7x96yA'
      map = new mapboxgl.Map({
        ...mapOptions,
        container: 'map'
      })

      map.on('load', () => {
        console.debug('map loaded')
      })

      map.on('mousemove', (e) => {
        mousePosition = e.lngLat
      })
    })

    watch(() => props.selectedLocation, (newLocation) => {
      if (newLocation && map) {
        const offset = [
          window.innerWidth * 0.2,
          0
        ]

        map.flyTo({
          center: newLocation.center,
          zoom: newLocation.zoom,
          duration: 1000,
          offset: offset,
          around: mousePosition || newLocation.center
        })
      }
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
.map-wrapper {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  overflow: hidden;
}

.map-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

/* 使用 CSS 变量控制地图容器的宽度和位置 */
.map-wrapper {
  padding-left: var(--sidebar-width);
  transition: padding-left 0.3s ease;
}

.map-wrapper:has(+ .sidebar-container.collapsed) {
  padding-left: 0;
}
</style> 
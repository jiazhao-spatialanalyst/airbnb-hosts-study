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
import axios from 'axios'

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
    },
    selectedHostTypes: {
      type: Array,
      default: () => []
    },
    currentTime: {
      type: Number,
      default: null
    },
    isHexMode: {
      type: Boolean,
      default: false
    }
  },
  setup(props) {
    let map = null
    let currentCenter = null
    let mousePosition = null
    let hexLayer = null
    const layerColors = {
      highly_commercial: '#FF0000',  // 红色
      commercial: '#FFA500',         // 橙色
      semi_commercial: '#FFFF00',    // 黄色
      dual_host: '#00FF00',         // 绿色
      single_host: '#0000FF'        // 蓝色
    }

    onMounted(() => {
      mapboxgl.accessToken = 'pk.eyJ1Ijoia3BmdWkiLCJhIjoiY2p6MWcxMXl4MDFlbTNsbDc1bnp6N3FjYSJ9.66qFOXwI661MOPOf7x96yA'
      map = new mapboxgl.Map({
        ...mapOptions,
        container: 'map'
      })

      map.on('load', () => {
        console.debug('map loaded')
        // 为每种类型创建图层
        Object.entries(layerColors).forEach(([type, color]) => {
          map.addSource(`listings-${type}`, {
            type: 'geojson',
            data: {
              type: 'FeatureCollection',
              features: []
            }
          })
          
          map.addLayer({
            id: `listings-layer-${type}`,
            type: 'circle',
            source: `listings-${type}`,
            paint: {
              'circle-radius': 3,
              'circle-color': color,
              'circle-opacity': 0.35
            }
          })
        })
      })

      map.on('mousemove', (e) => {
        mousePosition = e.lngLat
      })
    })

    const updateListings = async (hostType) => {
      console.log('Checking conditions:', {
        city: props.selectedLocation?.city,
        time: props.currentTime,
        type: hostType
      })

      if (!props.selectedLocation?.city || !props.currentTime) {
        console.log('Some conditions not met, skipping update')
        return
      }
      
      const date = new Date(Number(props.currentTime))
      const timeStr = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`
      
      try {
        const response = await axios.get(
          `http://localhost:8000/city/${props.selectedLocation.city}/listings_by_categories`,
          {
            params: {
              time_point: timeStr,
              categories: hostType
            },
            withCredentials: true,
            headers: {
              'Accept': 'application/json',
              'Content-Type': 'application/json'
            }
          }
        )
        
        if (!response.data || !response.data.listings) {
          console.error('Invalid response format:', response.data)
          return
        }
        
        const features = response.data.listings.map(listing => ({
          type: 'Feature',
          geometry: {
            type: 'Point',
            coordinates: [listing.longitude, listing.latitude]
          },
          properties: {
            host_id: listing.host_id,
            name: listing.name,
            price: listing.price
          }
        }))
        
        if (map.getSource(`listings-${hostType}`)) {
          map.getSource(`listings-${hostType}`).setData({
            type: 'FeatureCollection',
            features: features
          })
        }
      } catch (error) {
        console.error('Failed to fetch listings:', error)
      }
    }

    // 清除特定类型的图层数据
    const clearListings = (hostType) => {
      if (map.getSource(`listings-${hostType}`)) {
        map.getSource(`listings-${hostType}`).setData({
          type: 'FeatureCollection',
          features: []
        })
      }
    }

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

    // 监听房东类型和时间变化
    watch(
      [() => props.selectedHostTypes, () => props.currentTime],
      async (newValue, oldValue) => {
        if (!map) {
          console.log('Map not ready yet')
          return
        }
        
        const [newTypes, newTime] = newValue
        const [oldTypes, oldTime] = oldValue || [[], null]
        
        // 如果是网格图模式，直接更新网格
        if (props.isHexMode) {
          updateHexGrid()
          return
        }
        
        // 如果时间发生变化，更新所有已选类型的房源
        if (newTime !== oldTime) {
          console.log('Time changed, updating all selected types')
          for (const type of newTypes) {
            await updateListings(type)
          }
        } else {
          // 如果只是类型发生变化，则只处理变化的类型
          const oldSet = new Set(oldTypes || [])
          const newSet = new Set(newTypes)
          
          // 处理新增的类型
          for (const type of newSet) {
            if (!oldSet.has(type)) {
              await updateListings(type)
            }
          }
          
          // 处理移除的类型
          for (const type of oldSet) {
            if (!newSet.has(type)) {
              clearListings(type)
            }
          }
        }
      },
      { deep: true }
    )

    const updateHexGrid = async () => {
      if (!props.selectedLocation?.city || !props.currentTime) {
        return
      }
      
      try {
        const date = new Date(Number(props.currentTime))
        const timeStr = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`
        
        const response = await axios.get(
          `http://localhost:8000/city/${props.selectedLocation.city}/hexgrid`,
          {
            params: {
              time_point: timeStr,
              categories: props.selectedHostTypes.join(',')
            },
            withCredentials: true
          }
        )
        
        if (!map.getSource('hexgrid')) {
          map.addSource('hexgrid', {
            type: 'geojson',
            data: {
              type: 'FeatureCollection',
              features: []
            }
          })
          
          map.addLayer({
            id: 'hexgrid-layer',
            type: 'fill',
            source: 'hexgrid',
            paint: {
              'fill-color': [
                'interpolate',
                ['linear'],
                ['get', 'points_count'],
                0, 'rgba(0, 0, 255, 0)',
                1, 'rgba(0, 0, 255, 0.1)',
                10, 'rgba(0, 0, 255, 0.3)',
                50, 'rgba(0, 0, 255, 0.5)',
                100, 'rgba(0, 0, 255, 0.7)'
              ],
              'fill-opacity': 0.8
            }
          })
        }
        
        const features = response.data.hexagons.map(hex => ({
          type: 'Feature',
          properties: {
            id: hex.id,
            points_count: hex.points_count
          },
          geometry: {
            type: 'Polygon',
            coordinates: [
              hex.boundary.map(coord => [coord[1], coord[0]])
            ]
          }
        }))
        
        if (map.getLayer('hexgrid-layer')) {
          map.setLayoutProperty('hexgrid-layer', 'visibility', 'visible')
        }
        
        map.getSource('hexgrid').setData({
          type: 'FeatureCollection',
          features: features
        })
        
      } catch (error) {
        console.error('Failed to fetch hexgrid:', error)
      }
    }

    // 监听视图模式变化
    watch(() => props.isHexMode, (newMode) => {
      if (!map) return
      
      if (newMode) {
        // 隐藏散点图层
        Object.keys(layerColors).forEach(type => {
          if (map.getLayer(`listings-layer-${type}`)) {
            map.setLayoutProperty(`listings-layer-${type}`, 'visibility', 'none')
          }
        })
        // 立即更新网格
        updateHexGrid()
      } else {
        // 显示散点图层
        Object.keys(layerColors).forEach(type => {
          if (map.getLayer(`listings-layer-${type}`)) {
            map.setLayoutProperty(`listings-layer-${type}`, 'visibility', 'visible')
          }
        })
        // 隐藏网格图层
        if (map.getLayer('hexgrid-layer')) {
          map.setLayoutProperty('hexgrid-layer', 'visibility', 'none')
        }
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
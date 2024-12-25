<template>
  <div class="home-container">
    <MapView 
      :isSidebarCollapsed="isSidebarCollapsed"
      :selectedLocation="selectedLocation"
    />
    <Sidebar 
      :aboutToggle="aboutToggle" 
      @collapse-change="handleSidebarCollapse" 
      @city-selected="handleCitySelect"
      @time-changed="handleTimeChange"
    />
    <AboutButton 
      :aboutToggle="aboutToggle" 
      @toggle="aboutToggle = !aboutToggle" 
      class="z-20"
    />
    <LoadingSpinner v-if="loading" />
  </div>
</template>

<script>
import { ref } from 'vue'
import MapView from '../components/MapView.vue'
import Sidebar from '../components/Sidebar.vue'
import AboutButton from '../components/AboutButton.vue'
import LoadingSpinner from '../components/LoadingSpinner.vue'

export default {
  name: 'Home',
  components: {
    MapView,
    Sidebar,
    AboutButton,
    LoadingSpinner
  },
  setup() {
    const aboutToggle = ref(false)
    const loading = ref(false)
    const isSidebarCollapsed = ref(false)
    const selectedLocation = ref(null)

    const handleSidebarCollapse = (collapsed) => {
      isSidebarCollapsed.value = collapsed
    }

    const handleCitySelect = (location) => {
      selectedLocation.value = location
    }

    const handleTimeChange = (timestamp) => {
      console.log('Time changed:', new Date(timestamp))
      // 处理时间变化
    }

    return {
      aboutToggle,
      loading,
      isSidebarCollapsed,
      handleSidebarCollapse,
      selectedLocation,
      handleCitySelect,
      handleTimeChange
    }
  }
}
</script>

<style>
.home-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.loading-container {
  position: absolute;
  right: 0;
  top: 0;
  z-index: 10;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  margin: 1rem;
}
</style>

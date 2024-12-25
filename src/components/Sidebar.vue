<template>
  <div 
    id="sidebar" 
    class="sidebar-container transition-all duration-300"
    :class="{ 'collapsed': isCollapsed }"
  >
    <div class="sidebar-content" :class="{ 'opacity-0': isCollapsed }">
      <div class="title-box inline-flex" v-show="!aboutToggle">
        <h1 class="font-Roboto text-title text-bold">Airbnb Hosts Study</h1>
        <div class="ml-2 text-gray-75 text-detail self-end">
          v.{{ version }}
        </div>
      </div>

      <div v-show="!aboutToggle" class="controls-section mt-4">
        <div class="city-select mb-4">
          <label class="block text-sm font-medium text-gray-500 mb-2">Select City</label>
          <div class="relative">
            <select 
              v-model="selectedCity" 
              @change="onCityChange"
              class="w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
            >
              <option value="">Select a city below</option>
              <option v-for="city in cities" :key="city" :value="city">
                {{ city }}
              </option>
            </select>
          </div>
        </div>

        <div v-if="cityInfo" class="time-slider mb-4">
          <label class="block text-sm font-medium text-gray-500 mb-2">Time Range</label>
          <div class="time-range flex justify-between text-xs text-gray-500 mb-1">
            <span>{{ formatDate(timeRange[0]) }}</span>
            <span class="current-time">{{ formatDate(currentTime) }}</span>
            <span>{{ formatDate(timeRange[1]) }}</span>
          </div>
          <div class="relative pt-1">
            <input
              type="range"
              v-model="currentTime"
              :min="timeRange[0]"
              :max="timeRange[1]"
              :step="2629746000"
              class="
                w-full
                h-2
                rounded-lg
                appearance-none
                cursor-pointer
                range-slider
              "
              @input="onTimeChange"
            />
          </div>
        </div>

        <div v-if="cityInfo" class="host-types mb-4">
          <label class="block text-sm font-medium text-gray-500 mb-2">Host Type</label>
          <div class="space-y-2">
            <label v-for="(label, category) in hostTypeLabels" 
              :key="category" 
              class="flex items-center cursor-pointer"
            >
              <input 
                type="checkbox"
                v-model="selectedHostTypes"
                :value="category"
                class="checkbox-input"
              >
              <span class="ml-2 text-sm text-gray-600">{{ label }}</span>
            </label>
          </div>
        </div>
      </div>

      <About v-if="aboutToggle" />
    </div>

    <button 
      class="toggle-button"
      @click="toggleSidebar"
      :class="{ 'collapsed': isCollapsed }"
    >
      <span class="arrow-icon">
        {{ isCollapsed ? '❯' : '❮' }}
      </span>
    </button>
  </div>
</template>

<script>
import About from './About.vue'
import { computed, ref, onMounted, watch } from 'vue'
import { useStore } from 'vuex'
import axios from 'axios'

export default {
  name: 'Sidebar',
  components: {
    About
  },
  props: {
    aboutToggle: {
      type: Boolean,
      required: true
    }
  },
  setup(props, { emit }) {
    const store = useStore()
    const version = computed(() => store.state.version)
    const isCollapsed = ref(false)
    const cities = ref([])
    const selectedCity = ref('')
    const cityInfo = ref(null)
    const timeRange = ref([0, 0])
    const currentTime = ref(0)
    const selectedHostTypes = ref([])
    const hostTypeLabels = {
      highly_commercial: 'Highly Commercial',
      commercial: 'Commercial',
      semi_commercial: 'Semi-Commercial',
      dual_host: 'Dual Host',
      single_host: 'Single Host'
    }

    const toggleSidebar = () => {
      isCollapsed.value = !isCollapsed.value
      emit('collapse-change', isCollapsed.value)
    }

    const formatDate = (timestamp) => {
      const date = new Date(parseInt(timestamp))
      return date.toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'long'
      })
    }

    const fetchCities = async () => {
      try {
        const response = await axios.get('http://localhost:8000/cities', {
          withCredentials: true
        })
        cities.value = response.data.cities
      } catch (error) {
        console.error('Failed to fetch city list:', error)
      }
    }

    const onCityChange = async () => {
      if (!selectedCity.value) {
        cityInfo.value = null
        selectedHostTypes.value = []
        return
      }
      
      try {
        const response = await axios.get(`http://localhost:8000/city/${selectedCity.value}`, {
          withCredentials: true
        })
        cityInfo.value = response.data
        
        selectedHostTypes.value = []
        
        // Update time range
        timeRange.value = [
          new Date(cityInfo.value.time_window.earliest).getTime(),
          new Date(cityInfo.value.time_window.latest).getTime()
        ]
        currentTime.value = Number(timeRange.value[1])

        // Move the map to the selected city, adjust zoom level
        emit('city-selected', {
          city: selectedCity.value,
          center: [
            cityInfo.value.center.longitude,
            cityInfo.value.center.latitude
          ],
          zoom: 11
        })
      } catch (error) {
        console.error('Failed to fetch city data:', error)
        cityInfo.value = null
      }
    }

    const onTimeChange = () => {
      const timeValue = Number(currentTime.value)
      emit('time-changed', timeValue)
    }

    // 监听房东类型选择变化
    watch(selectedHostTypes, (newTypes) => {
      if (selectedCity.value && cityInfo.value) {
        emit('host-types-changed', newTypes)
      }
    })

    onMounted(() => {
      fetchCities()
    })

    return {
      version,
      isCollapsed,
      toggleSidebar,
      cities,
      selectedCity,
      cityInfo,
      timeRange,
      currentTime,
      onCityChange,
      onTimeChange,
      formatDate,
      selectedHostTypes,
      hostTypeLabels
    }
  }
}
</script>

<style>
.sidebar-container {
  width: var(--sidebar-width);
  min-width: var(--sidebar-width);
  position: fixed;
  left: 0;
  top: 0;
  height: 100%;
  background-color: white;
  font-family: Roboto, sans-serif;
  overflow: visible;
  box-shadow: 2px 0 10px rgba(0,0,0,0.1);
  transform: translateX(0);
  transition: transform 0.3s ease;
  z-index: 40;
}

.sidebar-container.collapsed {
  transform: translateX(-100%);
}

.sidebar-content {
  padding: 1rem;
  width: var(--sidebar-width);
  height: 100%;
  overflow-y: auto;
  transition: opacity 0.2s ease;
  background-color: white;
}

.toggle-button {
  position: absolute;
  right: -32px;
  top: 50%;
  transform: translateY(-50%);
  width: 32px;
  height: 64px;
  background-color: white;
  border: none;
  border-radius: 0 8px 8px 0;
  box-shadow: 2px 0 10px rgba(0,0,0,0.1);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 50;
}

.toggle-button:hover {
  background-color: #f0f0f0;
}

.toggle-button.collapsed {
  right: -32px;
  left: auto;
}

.arrow-icon {
  font-size: 20px;
  line-height: 1;
  color: #666;
  user-select: none;
}

/* 自定义下拉框样式 */
.custom-select {
  position: relative;
}

.select-input {
  appearance: none;
  background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3E%3Cpath stroke='%236B7280' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3E%3C/svg%3E");
  background-position: right 0.5rem center;
  background-repeat: no-repeat;
  background-size: 1.5em 1.5em;
  padding-right: 2.5rem;
}

.range-slider {
  -webkit-appearance: none;
  background: #e5e7eb;
  height: 4px;
  border-radius: 2px;
  outline: none;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  background: #ffffff;
  border: 2px solid #4f46e5;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  transition: all 0.2s ease;
  position: relative;
  z-index: 2;
}

.range-slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
}

.range-slider::-webkit-slider-thumb:active {
  transform: scale(1.1);
  background: #4f46e5;
  border-color: #4f46e5;
}

.range-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  background: #ffffff;
  border: 2px solid #4f46e5;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: all 0.2s ease;
}

.range-slider::-moz-range-thumb:hover {
  transform: scale(1.2);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.15);
}

.range-slider::-moz-range-thumb:active {
  transform: scale(1.1);
  background: #4f46e5;
  border-color: #4f46e5;
}

.range-slider::-moz-range-progress {
  background-color: #4f46e5;
  height: 6px;
  border-radius: 3px;
}

.range-slider::-moz-range-track {
  background-color: #e5e7eb;
  height: 6px;
  border-radius: 3px;
}

.time-slider {
  position: relative;
  padding-top: 24px;
}

.time-range {
  position: relative;
  padding: 0 8px;
}

.current-time {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  color: #4f46e5;
  font-weight: 500;
}

/* 自定义复选框样式 */
.checkbox-input {
  appearance: none;
  width: 16px;
  height: 16px;
  border: 2px solid #d1d5db;
  border-radius: 4px;
  margin-right: 8px;
  cursor: pointer;
  position: relative;
  background-color: white;
  transition: all 0.2s ease;
}

.checkbox-input:checked {
  background-color: #4f46e5;
  border-color: #4f46e5;
  background-image: url("data:image/svg+xml,%3csvg viewBox='0 0 16 16' fill='white' xmlns='http://www.w3.org/2000/svg'%3e%3cpath d='M12.207 4.793a1 1 0 010 1.414l-5 5a1 1 0 01-1.414 0l-2-2a1 1 0 011.414-1.414L6.5 9.086l4.293-4.293a1 1 0 011.414 0z'/%3e%3c/svg%3e");
  background-size: 100% 100%;
  background-position: center;
  background-repeat: no-repeat;
}

.checkbox-input:focus {
  outline: none;
  box-shadow: 0 0 0 2px #fff, 0 0 0 4px #4f46e5;
}

.host-types {
  padding: 8px 0;
}

.host-types label {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
  cursor: pointer;
}

.host-types label:last-child {
  margin-bottom: 0;
}
</style> 
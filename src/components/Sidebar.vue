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

      <div v-show="!aboutToggle" class="text-detail my-1">Analyzing Host Behavior in NYC</div>

      <div v-show="!aboutToggle" class="controls-section mt-4">
        <div class="city-select mb-4">
          <label class="block text-sm font-medium text-gray-700 mb-2">选择城市</label>
          <select 
            v-model="selectedCity" 
            @change="onCityChange"
            class="w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          >
            <option value="">请选择城市</option>
            <option v-for="city in cities" :key="city" :value="city">
              {{ city }}
            </option>
          </select>
        </div>

        <div v-if="cityInfo" class="time-slider mb-4">
          <label class="block text-sm font-medium text-gray-700 mb-2">
            时间范围
          </label>
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
import { computed, ref, onMounted } from 'vue'
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

    const toggleSidebar = () => {
      isCollapsed.value = !isCollapsed.value
      emit('collapse-change', isCollapsed.value)
    }

    const formatDate = (timestamp) => {
      const date = new Date(parseInt(timestamp))
      return date.toLocaleDateString('zh-CN', {
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
        console.error('获取城市列表失败:', error)
      }
    }

    const onCityChange = async () => {
      if (!selectedCity.value) {
        cityInfo.value = null
        return
      }
      
      try {
        const response = await axios.get(`http://localhost:8000/city/${selectedCity.value}`, {
          withCredentials: true
        })
        cityInfo.value = response.data
        
        // 更新时间范围
        timeRange.value = [
          new Date(cityInfo.value.time_window.earliest).getTime(),
          new Date(cityInfo.value.time_window.latest).getTime()
        ]
        currentTime.value = timeRange.value[1]

        // 移动地图到选中城市，调整缩放级别
        emit('city-selected', {
          center: [
            cityInfo.value.center.longitude,
            cityInfo.value.center.latitude
          ],
          zoom: 11
        })
      } catch (error) {
        console.error('获取城市数据失败:', error)
        cityInfo.value = null
      }
    }

    const onTimeChange = () => {
      emit('time-changed', currentTime.value)
    }

    const getTimePosition = () => {
      if (!timeRange.value[0] || !timeRange.value[1]) return 0
      const total = timeRange.value[1] - timeRange.value[0]
      const current = currentTime.value - timeRange.value[0]
      return (current / total) * 100
    }

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
      getTimePosition
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
</style> 
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

      <About v-if="aboutToggle" />

      <div v-show="!aboutToggle" id="analysis-section"></div>
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
import { computed, ref } from 'vue'
import { useStore } from 'vuex'

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

    const toggleSidebar = () => {
      isCollapsed.value = !isCollapsed.value
      emit('collapse-change', isCollapsed.value)
    }

    return {
      version,
      isCollapsed,
      toggleSidebar
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
</style> 
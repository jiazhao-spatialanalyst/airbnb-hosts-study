<template>
  <div
    id="about-toggle"
    class="about-button-container"
    @click="$emit('toggle')"
  >
    <div class="about-button text-Roboto text-detail font-bold">
      {{ aboutToggle === false ? 'About' : 'Analysis' }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'AboutButton',
  props: {
    aboutToggle: {
      type: Boolean,
      default: false
    }
  },
  emits: ['toggle']
}
</script>

<style>
.about-button-container {
  position: absolute;
  right: 15px;
  top: 15px;
  padding: 0.5rem;
  z-index: 10;
  border-radius: 0.375rem;
  transition: all 500ms ease-in-out;
  cursor: pointer;
  border: 1px solid;
  background-color: rgb(254, 240, 138);
}

.about-button-container:hover {
  background-color: rgb(253, 224, 71);
}
</style> 
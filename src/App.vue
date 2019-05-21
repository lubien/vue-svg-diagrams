<template>
  <div id="app" ref="parent">
    <svg :width="width" :height="height">
      <rect x="0" y="0" :width="width" :height="width" fill="#dedede"></rect>
      <Step :svg-width="width" :svg-height="height" :initial-x="500" :initial-y="200" draggable />
    </svg>
  </div>
</template>

<script>
import Step from './components/Step'

export default {
  name: 'app',

  components: { Step },

  data: () => ({
    width: 600,
    height: 4000
  }),

  mounted () {
    this.handleResize()
    window.addEventListener('resize', this.handleResize)
  },

  beforeDestroy () {
    window.removeEventListener('resize', this.handleResize)
  },

  methods: {
    handleResize () {
      const { width, height } = this.$refs.parent.getBoundingClientRect()
      this.width = width
      this.height = height
    }
  }
}
</script>

<style>
html, body, #app {
  padding: 0;
  margin: 0;
}

html, body {
  overflow: hidden;
}
</style>

<template>
  <div id="app" ref="parent">
    <svg :width="width" :height="height">
      <rect x="0" y="0" :width="width" :height="width" fill="#dedede"></rect>

      <Arrow
        v-if="Object.keys(positions).length"
        :id="4"
        :svg-width="width"
        :svg-height="height"
        :from="positions[1]"
        :to="positions[2]"
      />
      <Step :ref="`node:${1}`" @updatePosition="handleUpdatePosition" :id="1" :svg-width="width" :svg-height="height" :initial-x="50" :initial-y="200" draggable />
      <Step :ref="`node:${2}`" @updatePosition="handleUpdatePosition" :id="2" :svg-width="width" :svg-height="height" :initial-x="300" :initial-y="200" draggable />
      <Step :ref="`node:${3}`" @updatePosition="handleUpdatePosition" :id="3" :svg-width="width" :svg-height="height" :initial-x="550" :initial-y="200" draggable />
    </svg>
  </div>
</template>

<script>
import Step from './components/Step'
import Arrow from './components/Arrow'

export default {
  name: 'app',

  components: { Step, Arrow },

  data: () => ({
    width: 600,
    height: 4000,
    positions: {}
  }),

  mounted () {
    this.handleResize()
    this.handleUpdatePosition(1)
    this.handleUpdatePosition(2)
    this.handleUpdatePosition(3)
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
    },

    handleUpdatePosition (id) {
      const ref = this.$refs[`node:${id}`]
      this.positions = Object.assign({}, this.positions, { [id]: ref.getInfo() })
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

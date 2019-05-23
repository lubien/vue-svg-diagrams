<template>
  <div id="app" ref="parent">
    <svg :width="width" :height="height">
      <rect x="0" y="0" :width="width" :height="width" fill="#dedede"></rect>

      <Arrow
        v-for="edge in edges"
        :key="edge.id"
        :id="edge.id"
        :svg-width="width"
        :svg-height="height"
        :from="positions[edge.from]"
        :to="positions[edge.to]"
      />
      <Step
        v-for="node in nodes"
        :key="node.id"
        :ref="`node:${node.id}`"
        :id="node.id"
        :svg-width="width"
        :svg-height="height"
        :initial-x="node.x"
        :initial-y="node.y"
        draggable
        @updatePosition="handleUpdatePosition"
      />
    </svg>
  </div>
</template>

<script>
import Step from './Step'
import Arrow from './Arrow'

export default {
  name: 'Diagram',

  components: { Step, Arrow },

  props: {
    initialNodes: {
      type: Array,
      default: () => []
    },

    initialEdges: {
      type: Array,
      default: () => []
    }
  },

  data: () => ({
    width: 600,
    height: 4000,
    nodes: [],
    edges: [],
    positions: {}
  }),

  created () {
    for (let node of this.initialNodes) {
      this.nodes.push(Object.assign({}, node))
    }
  },

  mounted () {
    this.handleResize()

    for (let node of this.nodes) {
      this.handleUpdatePosition(node.id)
    }

    for (let edge of this.initialEdges) {
      this.edges.push(Object.assign({}, edge))
    }

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
      const ref = this.$refs[`node:${id}`][0]
      this.positions = Object.assign({}, this.positions, { [id]: ref.getInfo() })
    }
  }
}
</script>

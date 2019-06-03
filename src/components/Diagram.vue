<template>
  <div id="app" ref="parent">
    <svg class="main-svg" :width="width" :height="height">
      <rect
        x="0"
        y="0"
        :width="width"
        :height="width"
        fill="#dedede"
        class="bg-rect"
        @click="handleRectClick"
        @dblclick="handleDoubleRectClick"
      ></rect>

      <Arrow
        v-for="edge in edges"
        :key="`edge:${edge.id}`"
        :id="edge.id"
        :svg-width="width"
        :svg-height="height"
        :from="positions[edge.from]"
        :to="positions[edge.to]"
      />
      <Arrow
        v-if="nextEdgeFrom && (nextEdgeTo || mousePosition)"
        :key="`edge:new`"
        :svg-width="width"
        :svg-height="height"
        :from="nextEdgeFrom"
        :to="nextEdgeTo || {...mousePosition, width: 1, height: 1}"
      />
      <Step
        v-for="node in nodes"
        :key="`node:${node.id}`"
        :ref="`node:${node.id}`"
        :id="node.id"
        :svg-width="width"
        :svg-height="height"
        :initial-x="node.x"
        :initial-y="node.y"
        draggable
        @updatePosition="handleUpdatePosition"
        @createLink="handleCreateLink"
        @resize="handleStepResize"
        @click.native="handleStepClick()"
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
    positions: {},
    nextEdgeFromId: false,
    nextEdgeFrom: {},
    mousePosition: false
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
    window.removeEventListener('mousemove', this.onMouseMove)
  },

  computed: {
    nextEdgeToId () {
      if (!this.mousePosition) {
        return false
      }

      const { x, y } = this.mousePosition

      const found = this.nodes
        // reverse (prioritize newest nodes)
        .map((x, i, arr) => arr[arr.length - i - 1])
        .map(({ id }) => {
          const rect = this.positions[id]
          return rect && { id, rect }
        })
        .filter(x => x)
        .find(({ rect }) => isPointInside(rect, [x, y]))

      return found && found.id
    },

    nextEdgeTo () {
      return this.nextEdgeToId && this.positions[this.nextEdgeToId]
    }
  },

  methods: {
    handleResize () {
      const { width, height } = this.$refs.parent.getBoundingClientRect()
      this.width = width
      this.height = height
    },

    handleStepResize ({ id }) {
      this.handleUpdatePosition(id)
    },

    handleUpdatePosition (id) {
      const ref = this.$refs[`node:${id}`][0]
      this.positions = Object.assign({}, this.positions, { [id]: ref.getInfo() })
    },

    handleStepClick (stepId) {
      if (!this.nextEdgeFromId || !this.nextEdgeToId) {
        return
      }

      const highestId = Math.max(...this.edges.map(({ id }) => id))
      const id = this.edges.length ? highestId + 1 : 1
      const from = this.nextEdgeFromId
      const to = this.nextEdgeToId

      this.edges.push({
        id,
        from,
        to
      })

      this.nextEdgeFromId = false
      this.nextEdgeFrom = {}
    },

    handleRectClick () {
      this.nextEdgeFrom = false
      this.nextEdgeFromId = false
    },

    handleDoubleRectClick (e) {
      const id = 1 + this.nodes.map(({ id }) => id).sort().pop()
      const width = 200
      const height = 100

      const x = e.pageX - (width * 0.5)
      const y = e.pageY - (height * 0.5)

      this.nodes.push({
        id,
        x,
        y,
        width,
        height
      })

      // Wait for ref to be generated
      Promise.resolve().then(() => {
        this.handleUpdatePosition(id)
      })
    },

    handleCreateLink (id, from) {
      this.nextEdgeFromId = id
      this.nextEdgeFrom = from

      window.addEventListener('mousemove', this.onMouseMove)
    },

    onMouseMove (e) {
      if (!this.nextEdgeFrom) {
        return window.removeEventListener('mousemove', this.onMouseMove)
      }

      this.mousePosition = {
        x: e.pageX,
        y: e.pageY
      }
    }
  }
}

function isPointInside (rect, [x, y]) {
  const top = rect.y
  const bottom = top + rect.height
  const left = rect.x
  const right = left + rect.width

  const insideX = x >= left && x <= right
  const insideY = y >= top && y <= bottom

  return insideX && insideY
}
</script>

<style scoped>
.main-svg {
  user-select: none;
}

.bg-rect {
  cursor: pointer;
}
</style>

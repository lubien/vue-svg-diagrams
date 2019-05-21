<template>
  <svg :width="width" :height="height">
    <svg
      :x="x"
      :y="y"
      width="200"
      height="100"
      class="shadow"
      :class="{ draggable, dragging }"
      @mousedown="mouseDown"
    >
      <rect
        x="0"
        y="0"
        width="100%"
        height="100%"
        rx="5"
        ry="3"
        fill="white"
        stroke="#d9d9d9"
      ></rect>
      <a v-if="false" target="_blank">
        <text ref="text" x="50%" y="50%" fill="#34495e" font-family="Meiryo UI, sans-serif" font-size="20" text-anchor="middle">
          <tspan x="0" dy="0">Lorem ipsum dolor sit amet,</tspan>
          <tspan x="0" dy="30"> consectetur adipiscing elit.</tspan>
        </text>
      </a>

      <foreignObject ref="foreign" x="0" y="0" width="200" height="100">
        <div ref="div" xmlns="http://www.w3.org/1999/xhtml" class="foreign-div">
          Lorem ipsum
        </div>
      </foreignObject>
    </svg>
  </svg>
</template>

<script>
export default {
  name: 'Step',

  props: {
    width: {
      type: Number,
      required: true
    },

    height: {
      type: Number,
      required: true
    },

    initialX: {
      type: Number,
      required: true
    },

    initialY: {
      type: Number,
      required: true
    },

    draggable: {
      type: Boolean,
      default: false
    }
  },

  data: () => ({
    x: 0,
    y: 0,
    dragging: false,

    cursorOffset: {
      x: 0,
      y: 0
    },

    dragStart: {
      x: 0,
      y: 0
    }
  }),

  created () {
    this.x = this.initialX
    this.y = this.initialY
  },

  methods: {
    mouseDown (e) {
      if (!this.draggable) {
        return
      }

      this.cursorOffset.x = e.pageX
      this.cursorOffset.y = e.pageY
      this.dragStart.x = this.x
      this.dragStart.y = this.y
      this.dragging = true

      document.addEventListener('mousemove', this.mouseMove)
      document.addEventListener('mouseup', this.mouseUp)
    },

    mouseMove (e) {
      const x = e.pageX
      const y = e.pageY

      this.x = this.dragStart.x + (x - this.cursorOffset.x)
      this.y = this.dragStart.y + (y - this.cursorOffset.y)
    },

    mouseUp () {
      this.dragStart.x = null
      this.dragStart.y = null
      this.dragging = false

      document.removeEventListener('mousemove', this.mouseMove)
      document.removeEventListener('mouseup', this.mouseUp)
    }
  }
}
</script>

<style>
.foreign-div {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}

.draggable {
  cursor: grab;
}

.draggable.dragging {
  cursor: grabbing;
}
</style>

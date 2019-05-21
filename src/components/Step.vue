<template>
  <svg
    :width="svgWidth"
    :height="svgHeight"
    @blur="stopEditing"
    tabindex="-1"
    style="outline: none"
  >
    <g>
      <svg
        :x="x"
        :y="y"
        :width="width"
        :height="height"
        class="shadow"
        :class="{ draggable, dragging }"
        @mousedown="mouseDownDragging"
      >
        <rect
          x="0"
          y="0"
          :width="width"
          :height="height"
          rx="5"
          ry="3"
          fill="white"
          stroke="#d9d9d9"
        ></rect>

        <foreignObject ref="foreign" x="0" y="0" :width="width" :height="height">
          <div ref="div" xmlns="http://www.w3.org/1999/xhtml" class="foreign-div">
            Lorem ipsum
          </div>
        </foreignObject>
      </svg>
    </g>

    <g v-if="editing">
      <ellipse
        :x="x"
        :y="y"
        :cx="x"
        :cy="y"
        width="6"
        height="6"
        rx="6"
        ry="6"
        fill="#c6bde4"
        style="cursor: nw-resize"
        @mousedown="startResize({ fixBottom: true, fixRight: true })"
      ></ellipse>

      <ellipse
        :x="x"
        :y="y"
        :cx="x + width"
        :cy="y"
        width="6"
        height="6"
        rx="6"
        ry="6"
        fill="#c6bde4"
        style="cursor: ne-resize"
        @mousedown="startResize({ fixBottom: true, fixLeft: true })"
      ></ellipse>

      <ellipse
        :x="x"
        :y="y"
        :cx="x"
        :cy="y + height"
        width="6"
        height="6"
        rx="6"
        ry="6"
        fill="#c6bde4"
        style="cursor: sw-resize"
        @mousedown="startResize({ fixTop: true, fixRight: true })"
      ></ellipse>

      <ellipse
        :x="x"
        :y="y"
        :cx="x + width"
        :cy="y + height"
        width="6"
        height="6"
        rx="6"
        ry="6"
        fill="#c6bde4"
        style="cursor: se-resize"
        @mousedown="startResize({ fixTop: true, fixLeft: true })"
      ></ellipse>
    </g>
  </svg>
</template>

<script>
export default {
  name: 'Step',

  props: {
    svgWidth: {
      type: Number,
      required: true
    },

    svgHeight: {
      type: Number,
      required: true
    },

    initialWidth: {
      type: Number,
      default: 200
    },

    initialHeight: {
      type: Number,
      default: 100
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
    width: 200,
    height: 100,

    dragging: false,
    resizing: false,
    editing: false,

    cursorOffset: {
      x: 0,
      y: 0
    },

    dragStart: {
      x: 0,
      y: 0
    },

    fixedPositions: {
      top: 0,
      bottom: 0,
      left: 0,
      right: 0
    }
  }),

  created () {
    this.x = this.initialX
    this.y = this.initialY
    this.width = this.initialWidth
    this.height = this.initialHeight
  },

  mounted () {
    // this.resize()
  },

  methods: {
    mouseDownDragging (e) {
      this.editing = true

      if (!this.draggable) {
        return
      }

      this.cursorOffset.x = e.pageX
      this.cursorOffset.y = e.pageY
      this.dragStart.x = this.x
      this.dragStart.y = this.y
      this.dragging = true

      document.addEventListener('mousemove', this.mouseMoveDragging)
      document.addEventListener('mouseup', this.mouseUpDragging)
    },

    mouseMoveDragging (e) {
      const x = e.pageX
      const y = e.pageY

      this.x = this.dragStart.x + (x - this.cursorOffset.x)
      this.y = this.dragStart.y + (y - this.cursorOffset.y)
    },

    mouseUpDragging () {
      this.dragStart.x = null
      this.dragStart.y = null
      this.dragging = false

      document.removeEventListener('mousemove', this.mouseMoveDragging)
      document.removeEventListener('mouseup', this.mouseUpDragging)
    },

    startEditing () {
      this.editing = true
      document.body.addEventListener('click', this.stopEditing)
    },

    stopEditing () {
      this.editing = false
    },

    resize () {
      const textWidth = this.$refs.text.getComputedTextLength()
      const blockWidth = textWidth + 20
      this.blockWidth = blockWidth
    },

    startResize ({ fixTop = false, fixBottom = false, fixRight = false, fixLeft = false }) {
      this.fixedPositions.top = fixTop && this.y
      this.fixedPositions.bottom = fixBottom && this.y + this.height
      this.fixedPositions.right = fixRight && this.x + this.width
      this.fixedPositions.left = fixLeft && this.x
      this.resizing = true

      document.addEventListener('mousemove', this.mouseMoveResize)
      document.addEventListener('mouseup', this.mouseUpResizing)
    },

    mouseMoveResize (e) {
      const x = e.pageX
      const y = e.pageY

      if (this.fixedPositions.top) {
        this.height = Math.max(10, y - this.fixedPositions.top)
      }

      if (this.fixedPositions.bottom) {
        this.y = Math.max(0, y)
        this.height = Math.max(10, this.fixedPositions.bottom - y)
      }

      if (this.fixedPositions.right) {
        this.x = Math.max(0, x)
        this.width = Math.max(10, this.fixedPositions.right - x)
      }

      if (this.fixedPositions.left) {
        this.width = Math.max(10, x - this.fixedPositions.left)
      }
    },

    mouseUpResizing () {
      this.resizing = false

      document.removeEventListener('mousemove', this.mouseMoveResize)
      document.removeEventListener('mouseup', this.mouseUpResizing)
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

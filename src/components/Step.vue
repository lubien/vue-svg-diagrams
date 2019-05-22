<template>
  <svg
    :width="svgWidth"
    :height="svgHeight"
    style="outline: none"
  >
    <g ref="inside">
      <svg
        :x="x"
        :y="y"
        :width="width"
        :height="height"
        class="shadow"
        :class="{ draggable, dragging, 'content-editable': contentEditable }"
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
          <div
            ref="textDiv"
            xmlns="http://www.w3.org/1999/xhtml"
            class="foreign-div"
          >
            <span :contenteditable="contentEditable" class="node-text">Lorem ipsum</span>
          </div>
        </foreignObject>
      </svg>
    </g>

    <g v-if="showControls">
      <svg :x="x" :y="y - 30" :width="width" @mousedown="toggleContentEditable">
        <g>
          <ellipse
            :x="(width * 1/2) - 2"
            :y="12"
            :cx="(width * 1/2) - 2"
            :cy="12"
            rx="12"
            ry="12"
            fill="#4D4D4D"
          ></ellipse>

          <svg
            :x="(width * 1/2) - 9"
            :cx="(width * 1/2) - 9"
            :y="4"
            :cy="11"
            width="15"
            height="15"
            viewBox="0 0 1792 1792"
            xmlns="http://www.w3.org/2000/svg"
            fill="white"
          >
            <!-- Source: https://github.com/encharm/Font-Awesome-SVG-PNG/tree/master/black/svg -->
            <path
              v-if="contentEditable"
              d="M1490 1322q0 40-28 68l-136 136q-28 28-68 28t-68-28l-294-294-294 294q-28 28-68 28t-68-28l-136-136q-28-28-28-68t28-68l294-294-294-294q-28-28-28-68t28-68l136-136q28-28 68-28t68 28l294 294 294-294q28-28 68-28t68 28l136 136q28 28 28 68t-28 68l-294 294 294 294q28 28 28 68z"
            />
            <path
              v-else
              d="M491 1536l91-91-235-235-91 91v107h128v128h107zm523-928q0-22-22-22-10 0-17 7l-542 542q-7 7-7 17 0 22 22 22 10 0 17-7l542-542q7-7 7-17zm-54-192l416 416-832 832h-416v-416zm683 96q0 53-37 90l-166 166-416-416 166-165q36-38 90-38 53 0 91 38l235 234q37 39 37 91z"
            />
          </svg>
        </g>
      </svg>

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
const MIN_WIDTH = 100
const MIN_HEIGHT = 50

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
    showControls: false,
    contentEditable: false,

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
    this.width = Math.max(MIN_WIDTH, this.initialWidth)
    this.height = Math.max(MIN_HEIGHT, this.initialHeight)
  },

  methods: {
    mouseDownDragging (e) {
      this.displayControls()

      if (!this.draggable || this.contentEditable) {
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

    displayControls () {
      this.showControls = true
      document.body.addEventListener('mousedown', this.hideControls)
    },

    hideControls (e) {
      const clickIsOutside = e.target.contains(this.$el) || this.$el.contains(e.target)
      if (clickIsOutside || this.contentEditable) {
        return
      }
      this.showControls = false
      document.body.removeEventListener('click', this.hideControls)
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
        this.height = Math.max(MIN_HEIGHT, y - this.fixedPositions.top)
      }

      if (this.fixedPositions.bottom) {
        this.y = Math.max(0, y)
        this.height = Math.max(MIN_HEIGHT, this.fixedPositions.bottom - y)
      }

      if (this.fixedPositions.right) {
        this.x = Math.max(0, x)
        this.width = Math.max(MIN_WIDTH, this.fixedPositions.right - x)
      }

      if (this.fixedPositions.left) {
        this.width = Math.max(MIN_WIDTH, x - this.fixedPositions.left)
      }
    },

    mouseUpResizing () {
      this.resizing = false

      document.removeEventListener('mousemove', this.mouseMoveResize)
      document.removeEventListener('mouseup', this.mouseUpResizing)
    },

    toggleContentEditable (e) {
      e.preventDefault()
      e.stopPropagation()
      this.contentEditable = !this.contentEditable
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
  text-align: center;
}

.draggable {
  cursor: grab;
}

.draggable.dragging {
  cursor: grabbing;
  user-select: none;
}

.content-editable {
  cursor: text!important;
}

.node-text {
  width: 100%;
  outline: none;
}
</style>

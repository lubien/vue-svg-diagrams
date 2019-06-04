<template>
  <svg
    :width="svgWidth"
    :height="svgHeight"
    style="cursor: crosshair"
  >
    <defs>
      <marker id="arrow2" markerWidth="5" markerHeight="5" refX="8" refY="3" orient="auto" markerUnits="strokeWidth">
        <path d="M0,0 L0,6 L9,3 z" fill="#000" />
      </marker>

      <marker
        id="arrow"
        viewBox="0 0 10 10"
        refX="9"
        refY="5"
        markerWidth="6"
        markerHeight="6"
        orient="auto-start-reverse"
      >
        <path d="M 0 0 L 10 5 L 0 10 z" />
      </marker>
    </defs>

    <path :d="d" stroke="#000" stroke-width="3" fill="none" marker-end="url(#arrow)" />
  </svg>
</template>

<script>
export default {
  name: 'Arrow',

  props: {
    svgWidth: {
      type: Number,
      required: true
    },

    svgHeight: {
      type: Number,
      required: true
    },

    from: {
      type: Object,
      required: true
    },

    to: {
      type: Object,
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

  computed: {
    fromPoints () {
      return calculateNodePoints(this.from)
    },

    toPoints () {
      return calculateNodePoints(this.to)
    },

    d () {
      const allPairs = this.fromPoints
        .map(fromPoint =>
          this.toPoints.map(toPoint => [fromPoint, toPoint])
        )
        .reduce((acc, xs) => acc.concat(xs), [])

      const mapByDistance = allPairs.map(pair => ({
        distance: pointsDistance(...pair),
        pair
      }))

      const sorted = mapByDistance.sort((x, y) => x.distance - y.distance)

      const [[startX, startY], [endX, endY]] = sorted[0].pair

      return `M${startX},${startY} ${endX},${endY}`
    }
  }
}

function calculateNodePoints ({ x, y, width, height }) {
  const halfWidth = width * 0.5
  const halfHeight = height * 0.5

  const top = [x + halfWidth, y]
  const bottom = [x + halfWidth, y + height]
  const left = [x, y + halfHeight]
  const right = [x + width, y + halfHeight]

  return [top, bottom, left, right]
}

function pointsDistance ([x1, y1], [x2, y2]) {
  return Math.sqrt(
    Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2)
  )
}
</script>

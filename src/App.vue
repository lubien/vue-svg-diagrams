<template>
  <Diagram
    :initial-nodes="initialNodes"
    :initial-edges="initialEdges"
  />
</template>

<script>
import Diagram from './components/Diagram'

export default {
  name: 'app',

  components: { Diagram },

  data: () => ({
    initialNodes: [
      {
        id: 1,
        x: 50,
        y: 200
      },

      {
        id: 2,
        x: 500,
        y: 200
      },

      {
        id: 3,
        x: 350,
        y: 500
      }
    ],

    initialEdges: [
      {
        id: 1,
        from: 1,
        to: 2
      },

      {
        id: 2,
        from: 1,
        to: 3
      },

      {
        id: 3,
        from: 2,
        to: 3
      }
    ]
  })
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

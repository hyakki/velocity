<template>
  <div class="app">
    <Webgl class="app__webgl" />
    <div class="app__infos">
      <div class="app__infos__field velocity">Velocity: {{ velocity }}</div>
      <div class="app__infos__field acceleration">
        Acceleration: {{ acceleration }}
      </div>
      <div class="app__infos__field drag">Drag: {{ drag }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import Webgl from '@/components/Webgl/Webgl.vue'
import { ref } from 'vue'

import mitt from '@/mitt/mitt'

export default {
  name: 'App',
  components: {
    Webgl,
  },
  setup() {
    const velocity = ref(0)
    const acceleration = ref(0)
    const drag = ref(0)

    mitt.on('velocity', v => {
      velocity.value = v
    })

    mitt.on('acceleration', v => {
      acceleration.value = v
    })

    mitt.on('drag', v => {
      drag.value = v
    })

    return {
      velocity,
      acceleration,
      drag,
    }
  },
}
</script>

<style lang="scss">
[class*='app--'],
body {
  overscroll-behavior: none;
}

.app {
  @include get-all-space;

  color: $c-black;
}

.app__webgl {
  @include get-all-space;

  overflow: hidden;
}

.app__infos {
  position: absolute;
  z-index: 2;
  top: 10px;
  left: 10px;
  width: 200px;
  padding: $spacing;
  color: $c-white;
  border: 1px solid $c-gray-dark;
  background-color: $c-black;
}

.app__infos__field {
  @extend %fw-bold;

  font-size: 2rem;
  line-height: 1em;

  & + & {
    margin-top: $spacing / 2;
  }

  &.velocity {
    color: red;
  }

  &.acceleration {
    color: green;
  }

  &.drag {
    color: blue;
  }
}
</style>

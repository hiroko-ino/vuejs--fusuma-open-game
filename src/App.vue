<template>
  <div class="fullscreen" ref="demoBox">
    <div class="info">
      {{ count }}
    </div>
    <div
      v-for="fusuma in fusumas"
      :key="fusuma.id"
      class="fusuma-wrapper"
      :style="{ 'z-index': 10 - fusuma.id }"
    >
      <div
        class="fusuma-left"
        :style="{
          transform: `translateX(${getFusumaPosition(count, fusuma.id, 'left')}%)`,
          backgroundColor: fusuma.color
        }"
      />
      <div
        class="fusuma-right"
        :style="{
          transform: `translateX(${getFusumaPosition(count, fusuma.id, 'right')}%)`,
          backgroundColor: fusuma.color
        }"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { usePinch, useHover } from '@vueuse/gesture'
import { interpolate } from 'popmotion'

const fusumas = [
  {
    id: 1,
    color: '#FF0000'
  },
  {
    id: 2,
    color: '#00FFFF'
  },
  {
    id: 3,
    color: '#0000FF'
  }
]

const demoBox = ref()

const count = ref(0)

const mapper = interpolate([0, 2000], [0, 300], { clamp: true })

const getFusumaPosition = (count, index, position) => {
  if (count <= (index - 1) * 100) {
    return 0
  } else if (count < index * 100) {
    return position === 'left' ? -count + (index - 1) * 100 : count - (index - 1) * 100
  } else {
    return position === 'left' ? -100 : 100
  }
}

usePinch(
  ({ offset: [d], pinching }) => {
    count.value = mapper(d)
  },
  {
    domTarget: demoBox,
    eventOptions: {
      passive: true
    }
  }
)

useHover(
  ({ hovering }) => {
    if (!hovering) {
      window.removeEventListener('wheel', cancelEvent, { passive: false })
      document.removeEventListener('gesturestart', cancelEvent)
      document.removeEventListener('gesturechange', cancelEvent)
      return
    }

    window.addEventListener('wheel', cancelEvent, { passive: false })
    document.addEventListener('gesturestart', cancelEvent)
    document.addEventListener('gesturechange', cancelEvent)
  },
  {
    domTarget: demoBox
  }
)

// Disable viewport pinch zoom on whole app

const cancelEvent = (e) => e.preventDefault()
</script>

<style scoped>
.fullscreen {
  width: 100%;
  height: 100dvh;
  position: relative;
}

.fusuma-wrapper {
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  position: absolute;
  overflow: hidden;
}

.fusuma-left {
  position: absolute;
  left: 0;
  top: 0;
  width: 50%;
  height: 100%;
  border-right: 10px solid #000000;
  box-sizing: border-box;
}

.fusuma-right {
  position: absolute;
  right: 0;
  top: 0;
  width: 50%;
  height: 100%;
  border-left: 10px solid #000000;
  box-sizing: border-box;
}

.info {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 100;
}
</style>

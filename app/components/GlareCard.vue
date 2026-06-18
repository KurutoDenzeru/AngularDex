<template>
  <div ref="refElement"
    class="group relative overflow-hidden rounded-[var(--radius,6px)] transition-shadow will-change-transform"
    :class="props.class"
    @pointermove="handlePointerMove"
    @pointerenter="handlePointerEnter"
    @pointerleave="handlePointerLeave"
    :style="{
      perspective: '800px',
    }">
    <div
      class="pointer-events-none absolute inset-0 z-10 opacity-[var(--opacity,0)] transition-opacity duration-[var(--duration,300ms)] ease-[var(--easing,ease)]"
      :style="{
        background: 'radial-gradient(circle at var(--m-x, 50%) var(--m-y, 50%), rgb(255 255 255 / 0.25), transparent 80%)'
      }" />
    <div
      :style="{
        transform: 'rotateX(var(--r-x, 0deg)) rotateY(var(--r-y, 0deg))',
        transition: 'transform var(--duration, 300ms) var(--easing, ease)'
      }">
      <slot />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, onMounted } from 'vue'
  import { useTimeoutFn } from '@vueuse/core'

  interface GlareCardProps {
    class?: string
  }

  const props = defineProps<GlareCardProps>()

  const refElement = ref<HTMLElement | null>(null)
  const isPointerInside = ref(false)

  const state = ref({
    glare: { x: 50, y: 50 },
    background: { x: 50, y: 50 },
    rotate: { x: 0, y: 0 },
  })

  const setCssVars = () => {
    const el = refElement.value
    if (!el) return
    el.style.setProperty('--m-x', `${state.value.glare.x}%`)
    el.style.setProperty('--m-y', `${state.value.glare.y}%`)
    el.style.setProperty('--r-x', `${state.value.rotate.x}deg`)
    el.style.setProperty('--r-y', `${state.value.rotate.y}deg`)
    el.style.setProperty('--bg-x', `${state.value.background.x}%`)
    el.style.setProperty('--bg-y', `${state.value.background.y}%`)
  }

  function handlePointerMove(event: PointerEvent) {
    const rotateFactor = 0.4
    const rect = refElement.value?.getBoundingClientRect()
    if (rect) {
      const position = {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top,
      }
      const percentage = {
        x: (100 / rect.width) * position.x,
        y: (100 / rect.height) * position.y,
      }
      const delta = {
        x: percentage.x - 50,
        y: percentage.y - 50,
      }
      state.value.background.x = 50 + percentage.x / 4 - 12.5
      state.value.background.y = 50 + percentage.y / 3 - 16.67
      state.value.rotate.x = -(delta.x / 3.5) * rotateFactor
      state.value.rotate.y = (delta.y / 2) * rotateFactor
      state.value.glare.x = percentage.x
      state.value.glare.y = percentage.y
      setCssVars()
    }
  }

  function handlePointerEnter() {
    isPointerInside.value = true
    useTimeoutFn(() => {
      if (isPointerInside.value && refElement.value) {
        refElement.value.style.setProperty('--duration', '0s')
      }
    }, 300)
  }

  function handlePointerLeave() {
    isPointerInside.value = false
    if (refElement.value) {
      refElement.value.style.removeProperty('--duration')
      state.value.rotate = { x: 0, y: 0 }
      setCssVars()
    }
  }

  onMounted(() => {
    // initialize CSS vars
    if (refElement.value) {
      refElement.value.style.setProperty('--duration', '300ms')
      refElement.value.style.setProperty('--easing', 'ease')
      refElement.value.style.setProperty('--opacity', '0')
      // make default corner radius smaller (xs-like)
      refElement.value.style.setProperty('--radius', '6px')
      setCssVars()
    }
  })
</script>

<style scoped>
  .group {
    transform-style: preserve-3d;
  }

  .group > div:last-child {
    backface-visibility: hidden;
  }
</style>

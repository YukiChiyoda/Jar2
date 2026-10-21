<script setup lang="ts">
import { onMounted, onUnmounted, shallowRef } from 'vue';

const clouds = [
  { name: 'amber', depthX: 1 / 5, depthY: 1 / 30 },
  { name: 'rose', depthX: -1 / 3, depthY: 1 / 50 },
  { name: 'blue', depthX: 1 / 10, depthY: 1 / 5 },
  { name: 'mint', depthX: -1 / 10, depthY: -1 / 10 },
] as const;

const pointer = shallowRef({ x: 0, y: 0 });
let pointerX = 0;
let pointerY = 0;
let frameId: number | undefined;
let motionQuery: MediaQueryList | undefined;

function updatePointer() {
  pointer.value = { x: pointerX, y: pointerY };
  frameId = undefined;
}

function handlePointerMove(event: PointerEvent) {
  if (event.pointerType !== 'mouse' || !motionQuery?.matches) return;

  pointerX = event.clientX;
  pointerY = event.clientY;
  frameId ??= window.requestAnimationFrame(updatePointer);
}

function resetPointer() {
  if (frameId !== undefined) {
    window.cancelAnimationFrame(frameId);
    frameId = undefined;
  }

  pointer.value = { x: 0, y: 0 };
}

onMounted(() => {
  motionQuery = window.matchMedia(
    '(hover: hover) and (pointer: fine) and (prefers-reduced-motion: no-preference)',
  );
  motionQuery.addEventListener('change', resetPointer);
  window.addEventListener('pointermove', handlePointerMove, { passive: true });
  window.addEventListener('blur', resetPointer);
  document.documentElement.addEventListener('pointerleave', resetPointer);
});

onUnmounted(() => {
  resetPointer();
  motionQuery?.removeEventListener('change', resetPointer);
  window.removeEventListener('pointermove', handlePointerMove);
  window.removeEventListener('blur', resetPointer);
  document.documentElement.removeEventListener('pointerleave', resetPointer);
});
</script>

<template>
  <div
    class="cloud-background"
    aria-hidden="true"
    :style="{ '--pointer-x': `${pointer.x}px`, '--pointer-y': `${pointer.y}px` }"
  >
    <div
      v-for="cloud in clouds"
      :key="cloud.name"
      class="cloud"
      :class="`cloud--${cloud.name}`"
      :style="{ '--depth-x': cloud.depthX, '--depth-y': cloud.depthY }"
    ></div>
  </div>
</template>

<style scoped>
.cloud-background {
  position: fixed;
  z-index: -1;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
}

.cloud {
  position: absolute;
  border-radius: 50%;
  background-color: currentColor;
  opacity: 0.5;
  filter: blur(15px);
}

.cloud--amber {
  top: 10%;
  left: 3%;
  width: 120px;
  height: 120px;
  color: #fbedc9;
  box-shadow:
    130px -30px 5px -10px,
    50px -50px 0 0,
    60px 20px 0 0,
    120px 30px 0 -20px,
    170px 10px 0 -10px;
}

.cloud--rose {
  top: 12%;
  right: 5%;
  width: 150px;
  height: 150px;
  color: #fcb6d0;
  box-shadow:
    170px -50px 0 -10px,
    80px -30px 0 0,
    60px 10px 0 0,
    120px 30px 0 -20px,
    170px 10px 0 -10px;
}

.cloud--blue {
  bottom: 5%;
  left: 8%;
  width: 180px;
  height: 180px;
  color: #acbfea;
  box-shadow:
    -130px 20px 0 5px,
    -60px 50px 0 0,
    -60px -20px 0 0,
    -130px -20px 0 50px,
    -100px -15px 0 30px;
  opacity: 0.3;
  filter: blur(20px);
}

.cloud--mint {
  right: 8%;
  bottom: -2%;
  width: 120px;
  height: 120px;
  color: #ace0dd;
  box-shadow:
    100px -30px 0 -20px,
    80px -25px 0 0,
    25px 25px 0 0,
    120px 30px 0 -30px,
    70px 20px 0 -20px,
    120px 20px 0 20px;
}

@media (hover: hover) and (pointer: fine) and (prefers-reduced-motion: no-preference) {
  .cloud {
    transform: translate3d(
      calc(var(--pointer-x) * var(--depth-x)),
      calc(var(--pointer-y) * var(--depth-y)),
      0
    );
  }
}

@media (max-width: 768px) {
  .cloud--amber {
    top: 3%;
    left: -3%;
    width: 60px;
    height: 60px;
    box-shadow:
      75px -15px 3px -5px,
      25px -25px 0 0,
      30px 10px 0 0,
      60px 30px 0 -10px,
      85px 5px 0 -5px;
  }

  .cloud--rose {
    top: 8%;
    right: 10%;
    width: 75px;
    height: 75px;
    box-shadow:
      85px -25px 0 -5px,
      40px -15px 0 0,
      30px 5px 0 0,
      60px 15px 0 -10px,
      85px 5px 0 -5px;
  }

  .cloud--blue {
    bottom: 5%;
    left: 15%;
    width: 90px;
    height: 90px;
    box-shadow:
      -75px 10px 0 5px,
      -30px 25px 0 0,
      -30px -10px 0 0,
      -65px -30px 0 25px,
      -50px -10px 0 15px;
  }

  .cloud--mint {
    right: 10%;
    bottom: 10%;
    width: 60px;
    height: 60px;
    box-shadow:
      50px -15px 0 -10px,
      40px -20px 0 0,
      20px 10px 0 0,
      60px 15px 0 -15px,
      35px 10px 0 -10px,
      60px 10px 0 10px;
  }
}
</style>

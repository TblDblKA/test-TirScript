<script setup lang="ts">
import { ref, computed } from "vue";

type DotCoordinates = {
  x?: number;
  y?: number;
}

type Positions = {
  start: DotCoordinates;
  end: DotCoordinates;
}

const startPos = ref<DotCoordinates>({})
const position = ref<Positions>({ start: {}, end: {} })

const isCropped = computed<boolean>(() => !!(position.value.start.x && position.value.end.x))

function onMouseDown(event: MouseEvent) {
  startPos.value.x = event.offsetX;
  startPos.value.y = event.offsetY;
}

function onMouseUp(event: MouseEvent) {
  position.value.start.x = startPos.value.x;
  position.value.start.y = startPos.value.y;
  position.value.end.x = event.offsetX;
  position.value.end.y = event.offsetY;
  createCropper()
}

type CropperPosition = {
  top?: number;
  right?: number;
  bottom?: number;
  left?: number;
}

const cropperPosition = ref<CropperPosition>({ top: 0, right: 0, bottom: 0, left: 0 })

function createCropper() {
  cropperPosition.value.top = Math.min(position.value.start.y!, position.value.end.y!)
  cropperPosition.value.bottom = 500 - Math.max(position.value.start.y!, position.value.end.y!);
  cropperPosition.value.left = Math.min(position.value.start.x!, position.value.end.x!)
  cropperPosition.value.right = 800 - Math.max(position.value.start.x!, position.value.end.x!);
}

// function cropperPosition(offsetX: number = 0, offsetY: number = 0) {
//   if (!isCropped) return {}
//   // top left width height
//
//   return { top: top + 'px', bottom: bottom + 'px', left: left + 'px', right: right + 'px' }
// }

const isDragging = ref<boolean>(false);
const startDragPos = ref<DotCoordinates>({})

function onClipperMouseDown(event: MouseEvent) {
  isDragging.value = true;
  startPos.value.x = event.offsetX;
  startPos.value.y = event.offsetY;
}

function onClipperMouseMove(event: MouseEvent) {
  if (!isDragging) return
  cropperPosition.value.top += event.movementY
  cropperPosition.value.bottom += event.movementY
  cropperPosition.value.left += event.movementX
  cropperPosition.value.right += event.movementX
}
</script>

<template>
<div class="cropper-wrapper">
  <img
      class="image"
      :class="{ 'image-blurred': isCropped }"
      src="@/assets/cat.jpg"
      @mousedown.self="onMouseDown"
      @mouseup="onMouseUp"
      @mousemove.prevent
      draggable="false"
  >
  <img
      v-if="isCropped"
      class="image image-cropped"
      src="@/assets/cat.jpg"
      draggable="false"
      :style="{ 'clip-path': `inset(${cropperPosition.top}px ${cropperPosition.right}px ${cropperPosition.bottom}px ${cropperPosition.left}px)` }"
      @mousedown="onClipperMouseDown"
      @mousemove="onClipperMouseMove"
      @mouseup="isDragging = false"
  >
  <div v-if="isCropped" class="crop-box">
    <div class="crop-corner" :style="{ top: `${position.start.y! - 5}px`, left: `${position.start.x! - 5}px` }"></div>
    <div class="crop-corner" :style="{ top: `${position.start.y! - 5}px`, left: `${position.end.x! - 5}px` }"></div>
    <div class="crop-corner" :style="{ top: `${position.end.y! - 5}px`, left: `${position.start.x! - 5}px` }"></div>
    <div class="crop-corner" :style="{ top: `${position.end.y! - 5}px`, left: `${position.end.x! - 5}px` }"></div>
  </div>
</div>
</template>

<style scoped>
.cropper-wrapper {
  background-color: #fff;
  width: 800px;
  height: 500px;
  position: relative;
}
.image {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.image-blurred {
  filter: opacity(25%);
}

.image-cropped {
  position: absolute;
  top: 0;
  left: 0;
}



.crop-corner {
  width: 10px;
  height: 10px;
  border: 1px solid black;
  background-color: #fff;
  position: absolute;
}
</style>
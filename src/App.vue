<script setup lang="ts">
import { onMounted } from "vue";
let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D;

const numColumns = 20;
const numRows = 20;

function drawHorizontalLine(startX: number, endX: number, y: number) {
  ctx.beginPath();
  ctx.moveTo(startX, y);
  ctx.lineTo(endX, y);
  ctx.stroke();
}

function drawVerticalLine(startY: number, endY: number, x: number) {
  ctx.beginPath();
  ctx.moveTo(x, startY);
  ctx.lineTo(x, endY);
  ctx.stroke();
}

function drawImageScaled(img: HTMLImageElement, ctx: CanvasRenderingContext2D) {
  const canvas = ctx.canvas;
  const hRatio = canvas.width / img.width;
  const vRatio = canvas.height / img.height;
  const ratio = Math.min(hRatio, vRatio);
  const centerShift_x = (canvas.width - img.width * ratio) / 2;
  const centerShift_y = (canvas.height - img.height * ratio) / 2;
  const dw = img.width*ratio;
  const dh = img.height*ratio;
  ctx.clearRect(0,0,canvas.width, canvas.height);
  ctx.drawImage(img, 0,0, img.width, img.height,
      centerShift_x,centerShift_y,dw, dh);

  return {startX: centerShift_x, startY: centerShift_y, endX: dw + centerShift_x, endY: dh + centerShift_y};
}

onMounted(() => {
  canvas = document.getElementById("canvas") as HTMLCanvasElement;
  ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
  // const drawImageOnCanvas = (e: Event) => {
  //   ctx.drawImage(e.target as HTMLImageElement, 0, 0);
  // }
  canvas.width = 0.98*window.innerWidth;
  canvas.height = 0.98*window.innerHeight;
})

const drawImage = () => {
  const image = document.getElementById("mainImage") as HTMLImageElement;
  const imageCoords = drawImageScaled(image, ctx);
  console.log(imageCoords);

  for (let i = 1; i < numRows; i++) {
    const yCoord = i * ctx.canvas.height / numRows;
    drawHorizontalLine(imageCoords.startX, imageCoords.endX, yCoord);
  }

  for (let j = 1; j < numColumns; j++) {
    const xCoord = imageCoords.startX + j * (imageCoords.endX - imageCoords.startX) / numColumns;
    drawVerticalLine(imageCoords.startY, imageCoords.endY, xCoord);
  }
}



</script>

<template>
<div>
  <canvas id="canvas"></canvas>
  <img src="/painted.png" alt="" style="display: none" id="mainImage" @load="drawImage" />
</div>
</template>

<style scoped>
</style>

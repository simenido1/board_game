<script setup lang="ts">
import { onMounted } from "vue";
interface ImageCoords {
  startX: number;
  startY: number;
  endX: number;
  endY: number;
}

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D;
let imageCoords: ImageCoords;

const numColumns = 20;
const numRows = 20;

function getMousePosition(event: MouseEvent) {
  let x = event.clientX - imageCoords.startX;
  let y = event.clientY - imageCoords.startY;
  return {x, y};
}

function handleClick(event: MouseEvent) {
  const clickCoords = getMousePosition(event);
  const row = Math.ceil(clickCoords.y / (imageCoords.endY - imageCoords.startY) * numRows);
  const col = Math.ceil(clickCoords.x / (imageCoords.endX - imageCoords.startX) * numColumns);
  console.log(row, col);
}

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

function drawImageScaled(img: HTMLImageElement, ctx: CanvasRenderingContext2D): ImageCoords {
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
  canvas.width = 0.98 * window.innerWidth;
  canvas.height = 0.98 * window.innerHeight;

  ctx.font = "20px Arial";
  ctx.fillStyle = "white";
  ctx.textAlign = "center";
})

const drawImage = () => {
  const image = document.getElementById("mainImage") as HTMLImageElement;
 imageCoords = drawImageScaled(image, ctx);

  for (let i = 1; i < numRows; i++) {
    const yCoord = imageCoords.startY + i * (imageCoords.endY - imageCoords.startY) / numRows;
    drawHorizontalLine(imageCoords.startX, imageCoords.endX, yCoord);
  }

  for (let j = 1; j < numColumns; j++) {
    const xCoord = imageCoords.startX + j * (imageCoords.endX - imageCoords.startX) / numColumns;
    drawVerticalLine(imageCoords.startY, imageCoords.endY, xCoord);
  }

  for (let i = 1; i < numRows + 1; i++) {
    const yCoord = imageCoords.startY + (i-0.5) * (imageCoords.endY - imageCoords.startY) / numRows;
    ctx.textBaseline = "middle";
    ctx.fillText(i.toString(), imageCoords.startX, yCoord)
  }

  for (let j = 1; j < numColumns + 1; j++) {
    const xCoord = imageCoords.startX + (j-0.5) * (imageCoords.endX - imageCoords.startX) / numColumns;
    ctx.textBaseline = "top";
    ctx.fillText(j.toString(), xCoord, imageCoords.startY);
  }
}



</script>

<template>
<div>
  <canvas id="canvas" @click="handleClick"></canvas>
  <img src="/painted.png" alt="" style="display: none" id="mainImage" @load="drawImage" />
</div>
</template>

<style scoped>
</style>

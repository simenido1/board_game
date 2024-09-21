<script setup lang="ts">
import { onMounted, ref } from "vue";
import jsonData from './data/data.json';
import ModalComponent from "./components/ModalComponent.vue";

interface ImageCoords {
  startX: number;
  startY: number;
  endX: number;
  endY: number;
}

interface JsonData {
  row: number;
  col: number;
  type: string;
}

interface JsonPostData extends JsonData {
  name: string;
  lore: string;
  situation: string;
  var1?: string;
  var2?: string;
  var3?: string;
  image: string;
}

interface JsonTextData extends JsonData {
  text: string;
}

const cards: Map<number, Map<number, JsonPostData | JsonTextData>> = new Map();

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D;
let imageCoords: ImageCoords;
let circleRadius: number;

const numColumns = 20;
const numRows = 20;

const showModal = ref(false);
const activeCard = ref<JsonPostData | JsonTextData>();

function getCenterCoords(row: number, col: number) {
  const x_center = imageCoords.startX + (imageCoords.endX - imageCoords.startX) * (col - 0.5) / numColumns;
  const y_center = imageCoords.startY + (imageCoords.endY - imageCoords.startY) * (row - 0.5) / numRows;
  return {x: x_center, y: y_center};
}

function drawCircle(row: number, col: number) {
  const centerCoords = getCenterCoords(row, col);

  ctx.beginPath();
  ctx.arc(centerCoords.x, centerCoords.y, circleRadius, 0, 2 * Math.PI);
  ctx.fillStyle = "white";
  ctx.fill();
}

function getCard(row: number, col: number) {
  return cards.get(row)?.get(col);
}

function getMousePosition(event: MouseEvent) {
  let x = event.clientX - imageCoords.startX;
  let y = event.clientY - imageCoords.startY;
  return {x, y};
}

function handleClick(event: MouseEvent) {
  const clickCoords = getMousePosition(event);
  const row = Math.ceil(clickCoords.y / (imageCoords.endY - imageCoords.startY) * numRows);
  const col = Math.ceil(clickCoords.x / (imageCoords.endX - imageCoords.startX) * numColumns);

  const card = getCard(row, col);
  if (!card) {
    return;
  }

  activeCard.value = card;
  showModal.value = true;
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
})

const onImageLoad = () => {
  const image = document.getElementById("mainImage") as HTMLImageElement;
  imageCoords = drawImageScaled(image, ctx);

  circleRadius = 0.2 * Math.min((imageCoords.endX - imageCoords.startX) / numColumns, (imageCoords.endY - imageCoords.startY) / numRows);

  for (let i = 1; i < numRows; i++) {
    const yCoord = imageCoords.startY + i * (imageCoords.endY - imageCoords.startY) / numRows;
    drawHorizontalLine(imageCoords.startX, imageCoords.endX, yCoord);
  }

  for (let j = 1; j < numColumns; j++) {
    const xCoord = imageCoords.startX + j * (imageCoords.endX - imageCoords.startX) / numColumns;
    drawVerticalLine(imageCoords.startY, imageCoords.endY, xCoord);
  }

  ctx.font = `${Math.round(2 * circleRadius)}px Arial`;
  ctx.fillStyle = "white";
  ctx.textAlign = "center";

  console.log(ctx.font);

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

  jsonData.forEach(jsonCardRaw => {
    let colMap = cards.get(jsonCardRaw.row);

    if (!colMap) {
      colMap = new Map<number, JsonPostData | JsonTextData>();
      cards.set(jsonCardRaw.row, colMap);
    }

    colMap.set(jsonCardRaw.col, jsonCardRaw);

    drawCircle(jsonCardRaw.row, jsonCardRaw.col);
  });
}



</script>

<template>
<div>
  <modal-component :show="showModal" @hidden="showModal = false">
    <div v-if="activeCard?.type === 'text'">
      {{(activeCard as JsonTextData).text}}
    </div>
    <div v-if="activeCard?.type === 'post'">
      <div class="card p-0" style="white-space: pre-wrap;">
      <div class="card-header p-0">
        {{ (activeCard as JsonPostData).lore }}
      </div>
      <div class="row">
        <div class="col">
          <img :src="`/board_game/${(activeCard as JsonPostData).image}`" class="w-100" alt="">
          <h5 class="card-title text-center">{{ (activeCard as JsonPostData).name }}</h5>
        </div>
        <div class="col">
            <div class="card-header p-0">
              <strong>Ситуация:</strong> {{ (activeCard as JsonPostData).situation}}
            </div>
            <ul class="list-group list-group-flush">
              <li class="list-group-item ps-0" v-if="(activeCard as JsonPostData).var1">{{ (activeCard as JsonPostData).var1 }}</li>
              <li class="list-group-item ps-0" v-if="(activeCard as JsonPostData).var2">{{ (activeCard as JsonPostData).var2 }}</li>
              <li class="list-group-item ps-0" v-if="(activeCard as JsonPostData).var3">{{ (activeCard as JsonPostData).var3 }}</li>
            </ul>
        </div>
      </div>
      </div>
    </div>
  </modal-component>
  <canvas id="canvas" @click="handleClick"></canvas>
  <img src="/painted.png" alt="" style="display: none" id="mainImage" @load="onImageLoad" />
</div>
</template>

<style scoped>
</style>

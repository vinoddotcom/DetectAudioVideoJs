<template>
  <div class="flex w-full justify-center items-center flex-col">
    <h1>hello, world!</h1>
    <div class="flex relative justify-center items-center">
      <img ref="imageRef" crossorigin='anonymous'
        src="	https://cdn.pixabay.com/photo/2019/10/09/15/22/tunisia-4537624_960_720.jpg" width="960" height="60" />
      <canvas class="absolute" ref="canvasRef" width="960" height="640" />
    </div>
  </div>
</template>
<script setup lang="ts">
import * as faceapi from "face-api.js"

const imageRef = ref<HTMLImageElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);

async function handleImage() {
  console.log('loaded');
  if (!imageRef.value || !canvasRef.value) return;
  const detection = await faceapi.detectAllFaces(
    imageRef.value,
    new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceExpressions();

  canvasRef.value.innerHTML = faceapi.createCanvasFromMedia(imageRef.value).innerHTML;
  faceapi.matchDimensions(canvasRef.value, {
    width: 940,
    height: 650
  })

  const resized = faceapi.resizeResults(detection, {
    width: 940,
    height: 650
  })
  faceapi.draw.drawDetections(canvasRef.value, resized);
  faceapi.draw.drawFaceExpressions(canvasRef.value, resized);
  faceapi.draw.drawFaceLandmarks(canvasRef.value, resized);
  console.log(detection)
}

onMounted(() => {
  // console.log("Loading configuration");
  //  debugger;
  Promise.all([
    faceapi.nets.tinyFaceDetector.loadFromUri("models"),
    faceapi.nets.faceLandmark68Net.loadFromUri("models"),
    faceapi.nets.faceExpressionNet.loadFromUri("models"),
  ]).then((handleImage));
  // console.log("Loaded face")
});

</script>

<template>
    <div class="flex w-full justify-center items-center flex-col">
      <div class="mt-8 flex flex-col justify-center items-center gap-4">
        <h1 class="">hello, world!</h1>
       <button @click="estiMate" class="border px-2 py bg-green-400 shadow-sm text-white">Estimate</button>
      </div>
      <div class="flex justify-center items-center mt-4 relative">
        <video ref="videoRef" class="rotate-y-180" id="inputVideo" autoplay muted playsinline width="960" height="640"></video>
        <canvas id="canvas" class="absolute" ref="canvasRef" width="960" height="640" />
      </div>
    </div>
</template>
<script setup lang="ts">
  import * as faceapi from "face-api.js"

  const videoRef = ref<HTMLVideoElement | null>(null);
  const canvasRef = ref<HTMLCanvasElement | null>(null);
  const isLoaded = ref(false);
  const estiMation = ref<null | {height: number; distance: number; leftBrow: number; rightBrow: number}>(null)

  const detection =ref<null | faceapi.WithFaceExpressions<faceapi.WithFaceLandmarks<{
    detection: faceapi.FaceDetection;
}, faceapi.FaceLandmarks68>>[]>(null);

  async function handleVideo(): Promise<null> {
   // console.log("Video loaded");
    if(!canvasRef.value || !videoRef.value) return null;
    if(videoRef.value.paused || videoRef.value.ended || !isLoaded.value)
        return setTimeout(() => handleVideo()) as unknown as null
    // faceapi.matchDimensions(canvasRef.value, {
    //   width: 940,
    //   height: 650
    // }) () => props.items

     detection.value = await faceapi.detectAllFaces(videoRef.value, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceExpressions()
  
    // const resized = faceapi.resizeResults(detection, {
    //   width: 940,
    //   height: 650
    // })

    if (detection.value) {
          const dims =  faceapi.matchDimensions(canvasRef.value, { width: 940,height: 650 }, true);
        faceapi.draw.drawDetections(canvasRef.value, faceapi.resizeResults(detection.value, dims));
        faceapi.draw.drawFaceExpressions(canvasRef.value, faceapi.resizeResults(detection.value, dims));
        faceapi.draw.drawFaceLandmarks(canvasRef.value, faceapi.resizeResults(detection.value, dims));
      }
if(detection.value[0]) { 
    const landMarks = detection.value[0].landmarks;
    const jawOutline = landMarks.getJawOutline()
    const nose = landMarks.getNose()
    const mouth = landMarks.getMouth()
    const leftEye = landMarks.getLeftEye()
    const rightEye = landMarks.getRightEye()
    const leftEyeBrow = landMarks.getLeftEyeBrow()
    const rightEyeBrow = landMarks.getRightEyeBrow()
   
    // console.log({jawOutline, leftEye, rightEye, leftEyeBrow, nose, mouth, rightEyeBrow});

    const dist1 = faceapi.euclideanDistance([rightEye[3].x, rightEye[3].y], [jawOutline[16].x, jawOutline[16].y])
    // console.log({dist1});

    const dist2 = faceapi.euclideanDistance([leftEye[0].x, leftEye[0].y], [jawOutline[0].x, jawOutline[0].y]);

    if(Math.floor(dist1) !== Math.floor(dist2)) {
      if(Math.floor(dist1) < Math.floor(dist2)) console.log("Moved Left");
      else console.log("Moved Right");
    }


  if(estiMation.value) {
    let imageH = landMarks.imageHeight;
    let leftBrowCurrent = leftEyeBrow[0].x - jawOutline[0].x;
    let rightBrowCurrent = leftEyeBrow[4].x - jawOutline[16].x;
    const D  = faceapi.euclideanDistance([nose[0].x, nose[0].y], [nose[3].x, nose[3].y]);
    if(imageH > estiMation.value.height) {
      // % increase = [(New number – Original number)/Original number] x 100
      let increasePercentageNose = ((imageH - estiMation.value.height)/estiMation.value.height)*100;
      let distanceShouldBeOfNose = estiMation.value.distance * (increasePercentageNose/100) + imageH;
      if(estiMation.value.distance > distanceShouldBeOfNose) console.log("Moved Up")
    } else {
      let decreasePercentageNose = ((estiMation.value.height- imageH)/imageH)*100;
      let distanceShouldBeOfNose = imageH - estiMation.value.distance * (decreasePercentageNose/100);
      if(estiMation.value.distance > distanceShouldBeOfNose) console.log("Moved Up");
    }

    if(leftBrowCurrent > estiMation.value.leftBrow) {
      // % increase = [(New number – Original number)/Original number] x 100
      let increasePercentageLeftBrow = ((leftBrowCurrent - estiMation.value.leftBrow)/estiMation.value.leftBrow)*100;
      let distanceShouldBeOfLeftBrow = estiMation.value.leftBrow * (increasePercentageLeftBrow/100) + imageH;
      if(estiMation.value.leftBrow > distanceShouldBeOfLeftBrow) console.log("Moved Up")
    } else {
      let decreasePercentageLeftBrow = ((estiMation.value.height- leftBrowCurrent)/leftBrowCurrent)*100;
      let distanceShouldBeLeftBrow = leftBrowCurrent - estiMation.value.leftBrow * (decreasePercentageLeftBrow/100);
      if(estiMation.value.distance > distanceShouldBeLeftBrow) console.log("Moved Up");
    }

    if(leftBrowCurrent < estiMation.value.leftBrow || rightBrowCurrent < estiMation.value.rightBrow) {
      if(leftBrowCurrent > estiMation.value.leftBrow) {
       
      } else if(rightBrowCurrent > estiMation.value.rightBrow) {

      }
    }
  }

  }


      setTimeout(() => handleVideo())
     // console.log(detection);
      return null;
  };

  function estiMate() {
   if(!detection.value) return;
   if(detection.value[0]) {
    const landMarks = detection.value[0].landmarks
    const jawOutline = landMarks.getJawOutline();
    const leftEyeBrow = landMarks.getLeftEyeBrow()
    const rightEyeBrow = landMarks.getRightEyeBrow()
    let imageH = landMarks.imageHeight;
    const nose = landMarks.getNose()
    const D  = faceapi.euclideanDistance([nose[0].x, nose[0].y], [nose[3].x, nose[3].y])
    let leftBrow = leftEyeBrow[0].x - jawOutline[0].x;
    let rightBrow = rightEyeBrow[4].x - jawOutline[16].x;
    console.log({ height: imageH, distance: D, leftBrow, rightBrow });
    estiMation.value ={ height: imageH, distance: D, leftBrow, rightBrow }; 
   }
  }

  onMounted(() => {
    // console.log("Loading configuration");
    //  debugger;
    Promise.all([
      faceapi.nets.tinyFaceDetector.loadFromUri("models"),
      faceapi.nets.faceLandmark68Net.loadFromUri("models"),
      faceapi.nets.faceExpressionNet.loadFromUri("models"),
    ]).then(async () => {
        isLoaded.value = true;
        const stream = await navigator.mediaDevices.getUserMedia({ video: {} });
        if(videoRef.value) {
        videoRef.value.srcObject = stream;
        handleVideo();
    }
    });
  });
    // console.log("Loaded face")
</script>
<style>
#inputVideo, #canvas 
{
    transform: rotateY(180deg);
    -webkit-transform:rotateY(180deg); /* Safari and Chrome */
    -moz-transform:rotateY(180deg); /* Firefox */
}
</style>
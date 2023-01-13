<template>
    <div class="flex w-full justify-center items-center flex-col">
      <h1>hello, world!</h1>
      <div class="flex relative justify-center items-center">
        <img ref="imageRef" crossorigin='anonymous' id="inputImg"
          src="	https://images.pexels.com/photos/1222271/pexels-photo-1222271.jpeg" width="960" height="60" />
        <canvas class="absolute" ref="canvasRef" width="960" height="640" />
        <!-- 1 https://images.pexels.com/photos/839011/pexels-photo-839011.jpeg -->
        <!-- 2 https://images.pexels.com/photos/1222271/pexels-photo-1222271.jpeg -->
        <!-- 3 https://cdn.pixabay.com/photo/2019/10/09/15/22/tunisia-4537624_960_720.jpg -->
        <!-- <div id="personalCanvas"
        class="rounded-full h-10 w-10 bg-red-500 absolute top-[100] left-[100]"/> -->
      </div> 
    </div>
  </template>
  <script setup lang="ts">
  import * as faceapi from "face-api.js"
  
  const imageRef = ref<HTMLImageElement | null>(null);
  const canvasRef = ref<HTMLCanvasElement | null>(null);

  interface States { left: number; right: number; height: number; width: number; }

  const states = ref<States[]>([]);
  const M  = ref<null | number>(null);
  const BReight = ref<null | number>(null);
  const BLeft = ref<null | number>(null);
  
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
    faceapi.draw.drawFaceLandmarks(canvasRef.value, resized,);
    const landMarks = detection[0].landmarks;
    const jawOutline = landMarks.getJawOutline()
    const nose = landMarks.getNose()
    const mouth = landMarks.getMouth()
    const leftEye = landMarks.getLeftEye()
    const rightEye = landMarks.getRightEye()
    const leftEyeBrow = landMarks.getLeftEyeBrow()
    const rightEyeBrow = landMarks.getRightEyeBrow()
    
    console.log({jawOutline, leftEye, rightEye, leftEyeBrow, nose, mouth, rightEyeBrow});

    const dist1 = faceapi.euclideanDistance([rightEye[3].x, rightEye[3].y], [jawOutline[16].x, jawOutline[16].y])
    // console.log({dist1});

    const dist2 = faceapi.euclideanDistance([leftEye[0].x, leftEye[0].y], [jawOutline[0].x, jawOutline[0].y]);

    if(Math.floor(dist1) !== Math.floor(dist2)) {
      if(Math.floor(dist1) < Math.floor(dist2)) console.log("Moved Left");
      else console.log("Moved Right");
    }
   const forUpDown = faceapi.euclideanDistance([rightEyeBrow[0].x, rightEyeBrow[0].y], [jawOutline[16].x, jawOutline[16].y]);
   const forUpDown1 = faceapi.euclideanDistance([leftEyeBrow[3].x, leftEyeBrow[3].y], [jawOutline[0].x, jawOutline[0].y]);
   console.log({forUpDown, forUpDown1}, {height: landMarks.imageHeight, width: landMarks.imageWidth });
   states.value.push({left: forUpDown, right: forUpDown1, height: landMarks.imageHeight, width: landMarks.imageWidth})
  
    /* if(M.value && BLeft.value && BReight.value) {

   if(forUpDown !== (M.value*landMarks.imageHeight + BReight.value) || forUpDown1 !== (M.value*landMarks.imageHeight + BLeft.value)) {
    if(forUpDown > (M.value*landMarks.imageHeight + BReight.value) || forUpDown1 > (M.value*landMarks.imageHeight + BLeft.value)) {
      console.log("Moved Down");
    } else console.log("Moved Up");
   }


   } */
   
  } 

  /* function createEquation() {
    let averageM = 0;
   for (let i = 0; i < states.value.length; i+=1) {
    let m  = (states.value[i+1].width - states.value[i].width)/ (states.value[i+1].height - states.value[i].height);
    averageM += m;
   }
   let B = 0;
   let B1 = 0;
   const length = states.value.length;
   for (let i = 0; i < states.value.length; i+=1) {
    let b = states.value[i].left - (averageM/length * states.value[i].width);
    B += b;
    let b1 = states.value[i].right - (averageM/length * states.value[i].width);
    B1 += b1;
   }
   
   M.value = averageM/length;
   BLeft.value = B/length;
   BReight.value = B1/length;
  }
  */
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

  <style>
  #personalCanvas {
    top:100;
    left:100;
  }

  </style>
  
<script setup>
import {
  DrawingUtils,
  FilesetResolver,
  GestureRecognizer,
} from "@mediapipe/tasks-vision";

const hasWebcam = ref(null);
const data = ref(null);
onMounted(async () => {
  const video = document.querySelector("video");
  const canvas = document.querySelector("canvas");
  const ctx = canvas.getContext("2d");

  const vision = await FilesetResolver.forVisionTasks(
    // path/to/wasm/root
    "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm"
  );
  const gestureRecognizer = await GestureRecognizer.createFromOptions(vision, {
    baseOptions: {
      modelAssetPath:
        "https://storage.googleapis.com/mediapipe-tasks/gesture_recognizer/gesture_recognizer.task",
      delegate: "GPU",
    },
    runningMode: "VIDEO",
    numHands: 2,
  });

  hasWebcam.value = !!(
    navigator.mediaDevices && navigator.mediaDevices.getUserMedia
  );

  if (hasWebcam.value) {
    navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
      video.srcObject = stream;
      video.addEventListener("loadeddata", renderLoop);
    });
  }

  alert(hasWebcam.value);

  let lastVideoTime = -1;

  function processResult(result) {
    data.value = result;
    ctx.save();
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    const drawingUtils = new DrawingUtils(ctx);

    if (result.landmarks) {
      for (const landmarks of result.landmarks) {
        drawingUtils.drawConnectors(
          landmarks,
          GestureRecognizer.HAND_CONNECTIONS,
          { color: "#035E7B", lineWidth: 1 }
        );
        drawingUtils.drawLandmarks(landmarks, {
          color: "#035E7B",
          radius: 2,
          lineWidth: 1,
          fillColor: "#F4E76E",
        });
      }
    }

    console.log(result);

    ctx.restore();
  }

  function renderLoop() {
    if (video.currentTime !== lastVideoTime) {
      const gestureRecognitionResult = gestureRecognizer.recognizeForVideo(
        video,
        Date.now()
      );
      processResult(gestureRecognitionResult);
      lastVideoTime = video.currentTime;
    }

    requestAnimationFrame(renderLoop);
  }
});
</script>

<template>
  <section>
    <div>
      <h1>Hello World!</h1>
      <p v-if="hasWebcam">Camera is on</p>
    </div>
    <div id="row">
      <div id="video-container">
        <video autoplay playsinline></video>
        <canvas></canvas>
      </div>
      <div
        id="info"
        v-if="data"
        v-for="(hand, index) in data.gestures"
        :key="index"
      >
        <p>
          {{ data.handedness[index][0].displayName }} hand:
          {{ hand[0].categoryName }}
          {{ Math.round(hand[0].score * 10000) / 100 }}%
        </p>
      </div>
    </div>
  </section>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
}

section {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 100vw;
  height: 100vh;
  box-sizing: border-box;
}

p {
  font-size: xx-large;
}

#video-container {
  width: 960px;
  height: 720px;
}

#row {
  flex-direction: row;
}

video {
  position: absolute;
  transform: scaleX(-1);
  width: 960px;
  height: 720px;
}

canvas {
  position: absolute;
  width: 960px;
  height: 720px;
  transform: scaleX(-1);
  border: 1px solid black;
}
</style>

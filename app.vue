<script setup>
import { FilesetResolver, GestureRecognizer } from "@mediapipe/tasks-vision";

const hasWebcam = ref(null);
const video = ref(null);

onMounted(async () => {
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
      video.value.srcObject = stream;
      video.value.addEventListener("loadeddata", renderLoop);
    });
  }

  let lastVideoTime = -1;

  function renderLoop() {
    if (video.currentTime !== lastVideoTime) {
      const gestureRecognitionResult =
        gestureRecognizer.recognizeForVideo(video);
      processResult(gestureRecognitionResult);
      lastVideoTime = video.currentTime;
    }

    requestAnimationFrame(renderLoop);
  }
});
</script>

<template>
  <div>
    <h1>Hello World!</h1>
    <p v-if="hasWebcam">Camera is on</p>
    <canvas></canvas>
  </div>
</template>

<style>
canvas {
  border: 1px solid black;
}
</style>

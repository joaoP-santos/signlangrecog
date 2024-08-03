<script setup>
useHead({
  title: "LIBRAS Alphabet",
  meta: [
    { name: "description", content: "LIBRAS Alphabet" },
    { name: "viewport", content: "width=device-width, initial-scale=1.0" },
  ],
});

useSeoMeta({
  title: "LIBRAS Alphabet",
  ogTitle: "LIBRAS Alphabet",
  description: "A real-time Brazilian Sign Language (LIBRAS) recognizer",
  ogDescription: "A real-time Brazilian Sign Language (LIBRAS) recognizer",
});

import {
  DrawingUtils,
  FilesetResolver,
  GestureRecognizer,
} from "@mediapipe/tasks-vision";

const hasWebcam = ref(null);
const data = ref(null);
const alpha = ref([
  "A",
  "B",
  "C",
  "D",
  "E",
  "F",
  "G",
  "H",
  "I",
  "J",
  "K",
  "L",
  "M",
  "N",
  "O",
  "P",
  "Q",
  "R",
  "S",
  "T",
  "U",
  "V",
  "W",
  "X",
  "Y",
  "Z",
]);

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
      modelAssetPath: "/libras-alpha-v1.task",
      delegate: "GPU",
    },
    runningMode: "VIDEO",
    numHands: 2,
    minHandPresenceConfidence: 0.6,
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
  <section
    class="flex flex-col justify-center items-center w-screen h-screen box-border bg-blue-50"
  >
    <div
      id="header"
      class="text-center flex flex=col justify-center items-center"
    >
      <h1 class="font-bold text-2xl">LIBRAS Alphabet</h1>
      <p v-if="!hasWebcam" class="text-lg">Waiting for camera</p>
    </div>
    <div id="row" class="flex flex-row box-content gap-4">
      <div id="video-container" class="flex-1 w-[960px] h-[720px]">
        <video
          autoplay
          playsinline
          class="w-full h-full transform -scale-x-50"
        ></video>
        <canvas
          class="absolute w-full h-full -scale-x-100 border border-black"
        ></canvas>
      </div>
      <div id="info" class="flex flex-col w-[360px] flex-1">
        <p
          v-if="data"
          v-for="(hand, index) in data.gestures"
          :key="index"
          class="text-lg"
        >
          {{ data.handedness[index][0].displayName }} hand:
          {{ hand[0].categoryName }}
          {{ Math.round(hand[0].score * 10000) / 100 }}%
        </p>
        <div id="alpha" class="flex flex-row flex-wrap w-full">
          <span
            class="flex justify-center items-center bg-blue-500 text-white w-[50px] h-[50px]"
            v-for="(alpha, index) in alpha"
            @click="console.log(alpha)"
            :key="index"
            >{{ alpha }}</span
          >
        </div>
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
</style>

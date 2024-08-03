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
const displayLetter = ref(null);
const currentLetter = ref([null, null]);
const timers = ref([0, 0]);
const history = ref("");
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

  function getCam() {
    navigator.mediaDevices
      .getUserMedia({ video: true })
      .then((stream) => {
        video.srcObject = stream;
        video.addEventListener("loadeddata", renderLoop);
      })
      .catch((error) => {
        console.error(error.message);
        getCam();
      });
  }

  if (hasWebcam.value) {
    getCam();
  }

  let lastVideoTime = -1;
  function define(ind, gestureName) {
    currentLetter.value[ind] = gestureName;
    timers.value[ind] = 0;
  }
  function write(result) {
    for (var index = 0; index < 2; index++) {
      var gesture = result.gestures[index];

      let i = null;
      if (result.handedness[index]) {
        i = result.handedness[index][0].displayName == "Left" ? 0 : 1;
        gesture = result.gestures[index][0].categoryName;
      }

      if (currentLetter.value[i] == gesture) {
        timers.value[i] += 0.75;
        if (timers.value[i] >= 20) {
          if (history.value.length > 100)
            history.value = history.value.slice(1);

          history.value += gesture;
          define(i, gesture);
        }
      } else {
        define(i, gesture);
      }
    }
  }

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
      write(gestureRecognitionResult);
      processResult(gestureRecognitionResult);
      lastVideoTime = video.currentTime;
    }

    requestAnimationFrame(renderLoop);
  }
});
</script>

<template>
  <section
    class="flex flex-col justify-center items-center w-screen h-screen box-border"
  >
    <div
      id="header"
      class="text-center flex flex=col justify-center items-center"
    >
      <h1 class="font-bold text-2xl">LIBRAS Alphabet</h1>
      <p v-if="!hasWebcam" class="text-lg">Waiting for camera</p>
    </div>
    <div id="row" class="flex flex-row justify-center items-center gap-4">
      <div id="video-container" class="flex box-content">
        <video
          autoplay
          playsinline
          class="w-[960px] h-[720px] transform -scale-x-100"
        ></video>
        <canvas
          class="absolute w-[960px] h-[720px] -scale-x-100 border border-black"
        ></canvas>
      </div>
      <div id="info" class="flex flex-col w-[360px] h-[720px] flex-1 gap-12">
        <div>
          <p v-for="(letter, index) in currentLetter" :key="index">
            {{ ["Left", "Right"][index] }} hand: {{ letter || "None" }}
            <Bar :width="`${(timers[index] * 100) / 20}`" />
          </p>
        </div>
        <div id="alpha" class="flex flex-row flex-wrap w-full">
          <span
            class="flex justify-center items-center bg-blue-500 text-white w-[50px] h-[50px]"
            v-for="(alpha, index) in alpha"
            @mouseenter="displayLetter = alpha"
            @mouseleave="displayLetter = ''"
            :key="index"
            >{{ alpha }}</span
          >
        </div>

        <p class="max-w-full break-words">{{ history }}</p>

        <img
          v-if="displayLetter"
          :src="`/images/libras/${displayLetter}.${
            ['H', 'J', 'K', 'X', 'Z'].includes(displayLetter) ? 'jpg' : 'png'
          }`"
          class="w-48 self-center"
        />
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

p {
  font-size: 1.5rem;
}
</style>

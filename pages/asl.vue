<script setup>
useHead({
  title: "ASL Alphabet",
  meta: [
    { name: "description", content: "ASL Alphabet" },
    { name: "viewport", content: "width=device-width, initial-scale=1.0" },
  ],
});

useSeoMeta({
  title: "ASL Alphabet",
  ogTitle: "ASL Alphabet",
  description: "A real-time American Sign Language (ASL) alphabet recognizer",
  ogDescription: "A real-time American Sign Language (ASL) alphabet recognizer",
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
  "spc",
  "del",
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
      modelAssetPath: "/asl-alpha.task",
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
        alert(error.message);
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
        gesture = result.gestures[index][0].categoryName || "N";
      }

      if (currentLetter.value[i] == gesture) {
        timers.value[i] += 0.75;
        if (timers.value[i] >= 20) {
          if (gesture == "space") {
            history.value += " ";
          } else if (gesture == "del") {
            history.value = history.value.slice(0, -1);
          } else history.value += gesture;

          if (history.value.length > 100)
            history.value = history.value.slice(1);
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
    <div id="row" class="flex flex-row justify-center items-center gap-4">
      <div id="video-container" class="flex box-content">
        <video
          autoplay
          playsinline
          class="w-[720px] h-[540px] transform -scale-x-100"
        ></video>
        <canvas
          class="absolute w-[720px] h-[540px] -scale-x-100 border border-black"
        ></canvas>
      </div>
      <div id="info" class="flex flex-col w-[360px] h-[540px] flex-1">
        <h1 class="font-bold text-3xl">ASL Alphabet</h1>
        <div
          class="radio-inputs flex flex-wrap box-border border-blue-400 border-2 rounded-none w-full text-sm"
        >
          <NuxtLink class="radio flex-1 text-center" to="/libras">
            <input type="radio" name="LIBRAS" class="hidden" />
            <span
              class="flex cursor-pointer items-center justify-center border-0 py-2.5 text-gray-700 transition duration-150 ease-in-out"
              >LIBRAS</span
            >
          </NuxtLink>

          <label class="radio flex-1 text-center">
            <input type="radio" name="ASL" class="hidden" />
            <span
              class="bg-blue-400 text-white name flex cursor-pointer items-center justify-center border-0 py-2.5 transition duration-150 ease-in-out"
              >ASL</span
            >
          </label>
        </div>

        <p v-if="!hasWebcam" class="text-lg">Waiting for camera</p>
        <div>
          <p v-for="(letter, index) in currentLetter" :key="index">
            {{ ["Left", "Right"][index] }} hand: {{ letter || "None" }}
            <Bar :width="`${(timers[index] * 100) / 20}`" />
          </p>
        </div>
        <div id="alpha" class="flex flex-row flex-wrap w-full mt-4 mb-4">
          <span
            class="flex justify-center items-center bg-blue-500 text-white w-8 h-8"
            v-for="(alpha, index) in alpha"
            @mouseenter="displayLetter = alpha"
            @mouseleave="displayLetter = ''"
            :key="index"
            >{{ alpha }}</span
          >
        </div>
        <h1 class="text-2xl">History</h1>
        <p class="max-w-full break-words">{{ history }}</p>

        <img
          v-if="displayLetter"
          :src="`/images/asl/${displayLetter}.jpg`"
          class="w-48 self-center"
        />
        <div class="flex-grow"></div>

        <span class="justify-self-end self-end"
          >Made by
          <NuxtLink class="underline" to="https://github.com/joaoP-santos"
            >joaoP-santos</NuxtLink
          ></span
        >
      </div>
    </div>
  </section>
</template>

<script setup>
const props = defineProps(["modelAssetPath", "imagesDir"]);

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
      modelAssetPath: props.modelAssetPath,
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
  function write(result, time) {
    if (time == undefined) return;
    for (var index = 0; index < 2; index++) {
      var gesture = result.gestures[index];

      let i = null;
      if (result.handedness[index]) {
        i = result.handedness[index][0].displayName == "Left" ? 0 : 1;
        gesture = result.gestures[index][0].categoryName || "N";
      }

      if (currentLetter.value[i] == gesture) {
        timers.value[i] += time * 0.04;
        console.log(time);
        if (timers.value[i] >= 50) {
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
  let result = null;
  function renderLoop() {
    requestAnimationFrame(renderLoop);

    if (video.currentTime === lastVideoTime) return;
    const gestureRecognitionResult = gestureRecognizer.recognizeForVideo(
      video,
      Date.now()
    );
    write(gestureRecognitionResult);
    processResult(gestureRecognitionResult);
    lastVideoTime = video.currentTime;
    result = gestureRecognitionResult;
  }

  var start = Date.now();
  setInterval(() => {
    var delta = Date.now() - start; // milliseconds elapsed since start
    write(result, delta); // in seconds
    start = Date.now();
    console.log(delta);
  }, 10); // update about every second
});
</script>

<template>
  <section
    class="flex flex-col justify-center items-center w-screen h-screen box-border"
  >
    <div id="row" class="flex flex-row justify-center items-center gap-4">
      <div id="video-container" class="flex flex-col box-content">
        <video
          autoplay
          playsinline
          class="max-h-[75vh] max-w-[50vw] transform -scale-x-100"
        ></video>
        <img
          v-if="displayLetter"
          :src="`/images/${imagesDir}/${displayLetter}.jpg`"
          class="absolute w-[10vw] self-center shadow-lg shadow-black"
        />
      </div>
      <div id="info" class="flex flex-col w-[360px] h-[540px] flex-1">
        <h1 class="font-bold text-3xl"><slot></slot> Alphabet</h1>
        <canvas class="-scale-x-100 border border-blue-600"></canvas>

        <p v-if="!hasWebcam" class="text-lg">Waiting for camera</p>
        <div>
          <p v-for="(letter, index) in currentLetter" :key="index">
            {{ ["Left", "Right"][index] }} hand: {{ letter || "None" }}
            <Bar :width="`${(timers[index] * 100) / 50}`" />
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

        <div class="flex-grow"></div>

        <span class="justify-self-end self-end"
          >Made by
          <NuxtLink
            class="underline"
            to="https://www.linkedin.com/in/jo%C3%A3o-p-m-santos/"
            >joaoP-santos</NuxtLink
          ></span
        >
      </div>
    </div>
  </section>
</template>

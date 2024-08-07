<script setup>
const props = defineProps(["modelAssetPath", "imagesDir"]);

import {
  DrawingUtils,
  FilesetResolver,
  GestureRecognizer,
} from "@mediapipe/tasks-vision";

import Groq from "groq-sdk";

const groq = new Groq({
  apiKey: process.env.LLM_API_KEY,
  dangerouslyAllowBrowser: true,
});

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

const gestureRecognitionResult = ref(null);
const answer = ref(null);

const messages = ref([
  {
    role: "system",
    content:
      "Reply with a really short answer and with no punctuation, letters only.",
  },
]);

async function sendMessage() {
  console.log("im going");

  if (history.value.length == 0) return;
  messages.value.push({ role: "user", content: history.value });

  history.value = "";

  await groq.chat.completions
    .create({
      messages: messages.value,
      model: "llama3-8b-8192",
    })
    .then(
      (completion) => (answer.value = completion.choices[0]?.message?.content)
    )
    .catch((err) => alert(err.message));

  alert(answer.value);

  const showImages = setInterval(() => {
    displayLetter.value = answer.value.charAt(0).toUpperCase();
    answer.value = answer.value.slice(1);

    if (answer.value.length == 0) clearInterval(showImages);
  }, 1000);
}

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
  function renderLoop() {
    requestAnimationFrame(renderLoop);

    if (video.currentTime === lastVideoTime) return;
    gestureRecognitionResult.value = gestureRecognizer.recognizeForVideo(
      video,
      Date.now()
    );
    processResult(gestureRecognitionResult.value);
    lastVideoTime = video.currentTime;
  }

  var start = Date.now();
  setInterval(() => {
    var delta = Date.now() - start;
    write(gestureRecognitionResult.value, delta);
    start = Date.now();
  }, 10);
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
        <button @click="sendMessage()" class="bg-blue-500 text-white">
          Send
        </button>
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

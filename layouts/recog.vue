<script setup>
import { ref, onMounted, computed } from "vue";
import {
  DrawingUtils,
  FilesetResolver,
  GestureRecognizer,
} from "@mediapipe/tasks-vision";

const props = defineProps(["modelAssetPath", "imagesDir"]);

// Existing reactive variables
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
  "space",
  "del",
]);

// Game-related reactive variables
const targetWord = ref("masterpiece");
const score = ref(0);
const message = ref("");
const history = ref("");

// Introduced variable to track the current letter index
const currentLetterIndex = ref(0);

// Timers and current letters for each hand
const displayLetter = ref(null);
const currentLetter = ref([null, null]);
const timers = ref([0, 0]);

// Additional computed properties

// Compute the next letter to display
const nextLetterDisplay = computed(() => {
  const letter = targetWord.value.charAt(currentLetterIndex.value) || "";
  return letter.toUpperCase();
});

// Compute the image URL for the expected sign
const expectedLetterImage = computed(() => {
  const letter = targetWord.value.charAt(currentLetterIndex.value);
  if (letter) {
    return `/images/${props.imagesDir}/${letter.toUpperCase()}.jpg`;
  }
  return null;
});

onMounted(async () => {
  const video = document.querySelector("video");
  const canvas = document.querySelector("canvas");
  const ctx = canvas.getContext("2d");

  const vision = await FilesetResolver.forVisionTasks(
    "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm"
  );
  const gestureRecognizer = await GestureRecognizer.createFromOptions(vision, {
    baseOptions: {
      modelAssetPath: props.modelAssetPath,
      delegate: "GPU",
    },
    runningMode: "VIDEO",
    numHands: 2,
    numHandsToProcess: 2,
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
    if (time === undefined) return;

    for (let index = 0; index < 2; index++) {
      let gesture = result.gestures[index];
      let i = null;

      if (result.handedness[index]) {
        i = result.handedness[index][0].displayName === "Left" ? 0 : 1;
        gesture = result.gestures[index][0].categoryName || "N";
      }

      if (currentLetter.value[i] === gesture) {
        timers.value[i] += time * 0.04;
        if (timers.value[i] >= 50) {
          // Expected letter
          const expectedLetter = targetWord.value
            .charAt(currentLetterIndex.value)
            .toUpperCase();

          if (gesture.toUpperCase() === expectedLetter) {
            // Correct letter
            history.value += gesture.toUpperCase();
            score.value += 1;
            message.value = `Correct! You typed "${gesture.toUpperCase()}".`;

            currentLetterIndex.value += 1;

            // Reset timers
            timers.value = [0, 0];

            if (currentLetterIndex.value >= targetWord.value.length) {
              // Word completed
              message.value = `Congratulations! You've completed the word "${targetWord.value.toUpperCase()}" and earned ${
                score.value
              } points.`;

              // Reset for next word
              history.value = "";
              currentLetterIndex.value = 0;
              score.value = 0;

              // Optionally, set a new word
              // setNewWord();
            }
          } else {
            // Incorrect letter
            message.value = `Incorrect. Expected "${expectedLetter}", but you signed "${gesture.toUpperCase()}". Try again from the beginning.`;
            history.value = "";
            currentLetterIndex.value = 0;
            timers.value = [0, 0];
          }

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

  let start = Date.now();
  setInterval(() => {
    const delta = Date.now() - start; // milliseconds elapsed since start
    write(result, delta);
    start = Date.now();
  }, 10); // update about every 10 milliseconds
});
</script>

<template>
  <section
    class="flex flex-col justify-center items-center w-screen h-screen box-border"
  >
    <div
      id="row"
      class="flex flex-col md:flex-row justify-center items-center gap-4"
    >
      <!-- Video Container -->
      <div
        id="video-container"
        class="flex flex-col box-content relative w-80 h-80 overflow-hidden rounded"
      >
        <video
          autoplay
          playsinline
          class="w-full h-full object-cover transform -scale-x-100"
        ></video>
        <!-- Display the expected sign image -->
        <img
          v-if="expectedLetterImage"
          :src="expectedLetterImage"
          alt="Expected Sign"
          class="absolute top-4 right-4 w-24 h-24 shadow-lg shadow-black rounded-full overflow-hidden"
        />
      </div>

      <!-- Info and Game Display -->
      <div
        id="info"
        class="flex flex-col w-[360px] h-[540px] flex-1 text-white"
      >
        <!-- Game Instructions -->
        <h1 class="font-bold text-3xl mb-4 text-center">
          Type the word "<span class="text-green-400">{{
            targetWord.toUpperCase()
          }}</span
          >" using sign language!
        </h1>

        <!-- Next Letter to Sign -->
        <p class="text-xl mb-2">
          Next Letter:
          <span class="font-semibold text-green-400">{{
            nextLetterDisplay
          }}</span>
        </p>

        <!-- User Input and Score -->
        <p class="text-xl mb-2">
          Your Input: <span class="font-semibold">{{ history }}</span>
        </p>
        <p class="text-xl mb-4">
          Score: <span class="font-semibold">{{ score }}</span>
        </p>

        <!-- Message Display -->
        <p v-if="message" class="text-yellow-400 text-lg mb-4">{{ message }}</p>

        <!-- Current Recognized Letters and Timers -->
        <div class="mb-4">
          <p v-for="(letter, index) in currentLetter" :key="index">
            {{ ["Left", "Right"][index] }} hand:
            <span class="font-semibold">{{ letter || "None" }}</span>
            <Bar :width="(timers[index] * 100) / 50" />
          </p>
        </div>

        <!-- Canvas for Drawing Landmarks -->
        <canvas
          class="-scale-x-100 border border-blue-600 rounded-full w-80 h-80"
        ></canvas>

        <!-- Alphabet Reference -->
        <div id="alpha" class="flex flex-row flex-wrap w-full mt-4 mb-4">
          <span
            class="flex justify-center items-center bg-green-500 text-white w-8 h-8 m-1 cursor-pointer rounded-full"
            v-for="(letter, index) in alpha"
            @mouseenter="displayLetter = letter"
            @mouseleave="displayLetter = ''"
            :key="index"
            >{{ letter }}</span
          >
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* Add any custom styles here */
/* You can include Duolingo-like font styles or additional theming if desired */
</style>

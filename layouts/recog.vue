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
  <Navbar />
  <section
    class="flex flex-col justify-center mt-[10vh] h-[90vh] items-center w-screen box-border"
  >
    <div
      id="row"
      class="flex flex-col md:flex-row px-24 justify-center items-center gap-4"
    >
      <!-- Video Container -->
      <div
        id="video-container"
        class="flex flex-col relative overflow-hidden flex-1 rounded"
      >
        <video
          autoplay
          playsinline
          class="w-full h-auto object-cover transform -scale-x-100"
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
      <div id="info" class="flex flex-col w-full flex-1">
        <!-- Game Instructions -->
        <h1 class="font-medium text-2xl mb-4">
          Digite a palavra
          <span class="text-blue">{{ targetWord.toUpperCase() }}</span> usando
          línguas de sinais!
        </h1>

        <!-- Next Letter to Sign -->
        <p class="text-xl mb-2">
          Próxima letra:
          <span class="font-semibold text-blue">{{ nextLetterDisplay }}</span>
        </p>

        <!-- User Input and Score -->
        <p v-if="history" class="text-xl mb-2">
          Progresso: <span class="font-semibold">{{ history }}</span>
        </p>
        <p v-if="score" class="text-xl mb-4">
          Pontuação: <span class="font-semibold">{{ score }}</span>
        </p>

        <!-- Message Display -->
        <p v-if="message" class="text-sunglow text-lg mb-4">{{ message }}</p>

        <!-- Current Recognized Letters and Timers -->
        <div class="mb-4">
          <p v-for="(letter, index) in currentLetter" :key="index">
            {{ ["Left", "Right"][index] }} hand:
            <span class="font-semibold">{{ letter || "None" }}</span>
            <Bar :width="(timers[index] * 100) / 50" />
          </p>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* Add any custom styles here */
/* You can include Duolingo-like font styles or additional theming if desired */
</style>

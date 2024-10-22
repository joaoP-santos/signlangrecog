<!-- components/recog.vue -->
<template>
  <div>
    <!-- Navbar Component -->
    <Navbar />

    <!-- Main Section -->
    <section
      class="flex flex-col justify-center mt-[10vh] h-[90vh] items-center w-screen box-border"
    >
      <div
        id="row"
        class="flex flex-col md:flex-row px-24 justify-center items-center gap-4"
      >
        <!-- Video and Canvas Container -->
        <div
          id="video-container"
          class="flex flex-col relative overflow-hidden flex-1 rounded"
        >
          <!-- Video Element -->
          <video
            ref="video"
            autoplay
            playsinline
            class="w-full h-auto object-cover transform -scale-x-100"
          ></video>

          <!-- Canvas for Drawing Annotations -->
          <canvas
            ref="canvas"
            class="absolute top-0 left-0 w-full h-full pointer-events-none"
          ></canvas>

          <!-- Expected Sign Image -->
          <img
            v-if="expectedLetterImage"
            :src="expectedLetterImage"
            alt="Expected Sign"
            class="absolute top-4 right-4 w-24 h-24 shadow-lg shadow-black rounded-full overflow-hidden"
          />
        </div>

        <!-- Information and Game Display -->
        <div id="info" class="flex flex-col w-full flex-1">
          <!-- Game Instructions -->
          <h1 class="font-medium text-2xl mb-4">
            Sign the letter
            <span class="text-blue">{{ targetLetter.toUpperCase() }}</span> using
            sign language!
          </h1>

          <!-- Current Letter to Sign -->
          <p class="text-xl mb-2">
            Current letter:
            <span class="font-semibold text-blue">{{ targetLetter.toUpperCase() }}</span>
          </p>

          <!-- User Score -->
          <p v-if="score" class="text-xl mb-4">
            Score: <span class="font-semibold">{{ score }}</span>
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
  </div>
</template>

<script setup>
// Import necessary functions and components
import { ref, onMounted, computed } from "vue";
import {
  DrawingUtils,
  FilesetResolver,
  GestureRecognizer,
} from "@mediapipe/tasks-vision";

// Importing Custom Components
import Navbar from "~/components/Navbar.vue";
import Bar from "~/components/Bar.vue";

// Define Props with Validation
const props = defineProps({
  modelAssetPath: {
    type: String,
    required: true,
  },
  imagesDir: {
    type: String,
    required: true,
  },
});

// Function to Get a Random Letter
function getRandomLetter() {
  const letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
  return letters.charAt(Math.floor(Math.random() * letters.length));
}

// Reactive Variables
const hasWebcam = ref(false);
const data = ref(null);
const alpha = ref([
  "A", "B", "C", "D", "E", "F", "G", "H", "I", "J",
  "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T",
  "U", "V", "W", "X", "Y", "Z", "space", "del",
]);

// Game-related Reactive Variables
const targetLetter = ref(getRandomLetter());
const score = ref(0);
const message = ref("");

// Timers and Current Letters for Each Hand
const currentLetter = ref([null, null]);
const timers = ref([0, 0]);

// Computed Properties
const expectedLetterImage = computed(() => {
  const letter = targetLetter.value;
  if (letter) {
    return `/images/${props.imagesDir}/${letter.toUpperCase()}.jpg`;
  }
  return null;
});

// References to Video and Canvas Elements
const video = ref(null);
const canvas = ref(null);

onMounted(async () => {
  const videoElement = video.value;
  const canvasElement = canvas.value;

  if (!videoElement || !canvasElement) {
    console.error("Video or Canvas element not found.");
    return;
  }

  const ctx = canvasElement.getContext("2d");

  if (!ctx) {
    console.error("Canvas context could not be obtained.");
    return;
  }

  try {
    // Initialize Mediapipe Tasks Vision
    const vision = await FilesetResolver.forVisionTasks(
      "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm"
    );

    const gestureRecognizer = await GestureRecognizer.createFromOptions(
      vision,
      {
        baseOptions: {
          modelAssetPath: props.modelAssetPath,
          delegate: "GPU",
        },
        runningMode: "VIDEO",
        numHands: 2,
        numHandsToProcess: 2,
        minHandPresenceConfidence: 0.6,
      }
    );

    // Check for Webcam Availability
    hasWebcam.value =
      !!(navigator.mediaDevices && navigator.mediaDevices.getUserMedia);

    // Function to Access Webcam
    function getCam() {
      navigator.mediaDevices
        .getUserMedia({ video: true })
        .then((stream) => {
          videoElement.srcObject = stream;
          videoElement.addEventListener("loadeddata", renderLoop);
        })
        .catch((error) => {
          alert(`Error accessing webcam: ${error.message}`);
          console.error("Webcam access error:", error);
        });
    }

    if (hasWebcam.value) {
      getCam();
    } else {
      alert("Webcam not available on this device.");
      console.error("Webcam not available.");
    }

    let lastVideoTime = -1;
    let result = null;

    // Function to Define Current Letter
    function define(ind, gestureName) {
      currentLetter.value[ind] = gestureName;
      timers.value[ind] = 0;
    }

    // Function to Handle Gesture Recognition Results
    function write(resultData, time) {
      if (time === undefined) return;

      for (let index = 0; index < 2; index++) {
        let gesture = resultData.gestures[index];
        let handIndex = null;

        if (resultData.handedness[index]) {
          // Invert Handedness Mapping Due to Mirrored Video
          // If Mediapipe detects "Left", it's actually the right hand on the screen
          handIndex =
            resultData.handedness[index][0].displayName === "Left" ? 1 : 0;
          gesture = resultData.gestures[index][0].categoryName || "N";
        }

        if (currentLetter.value[handIndex] === gesture) {
          timers.value[handIndex] += time * 0.04;
          if (timers.value[handIndex] >= 50) {
            // Expected Letter
            const expectedLetter = targetLetter.value.toUpperCase();

            if (gesture.toUpperCase() === expectedLetter) {
              // Correct Letter
              score.value += 1;
              message.value = `Correct! You signed "${gesture.toUpperCase()}".`;

              // Set a new letter
              targetLetter.value = getRandomLetter();

              // Reset Timers and Current Letters
              timers.value = [0, 0];
              currentLetter.value = [null, null];
            } else {
              // Incorrect Letter
              message.value = `Incorrect. Expected "${expectedLetter}", but you signed "${gesture.toUpperCase()}". Try again.`;
              // Reset Timers and Current Letters
              timers.value = [0, 0];
              currentLetter.value = [null, null];
            }

            define(handIndex, gesture);
          }
        } else {
          define(handIndex, gesture);
        }
      }
    }

    // Function to Process Recognition Results and Draw on Canvas
    function processResult(resultData) {
      data.value = resultData;
      ctx.save();

      // Flip the canvas horizontally to match the mirrored video
      ctx.scale(-1, 1);
      ctx.translate(-canvasElement.width, 0);

      ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);

      const drawingUtils = new DrawingUtils(ctx);

      if (resultData.landmarks) {
        for (const landmarks of resultData.landmarks) {
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

    // Render Loop to Continuously Process Video Frames
    function renderLoop() {
      requestAnimationFrame(renderLoop);

      if (videoElement.currentTime === lastVideoTime) return;

      const gestureRecognitionResult = gestureRecognizer.recognizeForVideo(
        videoElement,
        Date.now()
      );
      write(gestureRecognitionResult);
      processResult(gestureRecognitionResult);
      lastVideoTime = videoElement.currentTime;
      result = gestureRecognitionResult;
    }

    let start = Date.now();

    // Interval to Update Timers and Handle Gesture Recognition Over Time
    setInterval(() => {
      const delta = Date.now() - start; // Milliseconds Elapsed Since Start
      write(result, delta);
      start = Date.now();
    }, 10); // Update Every 10 Milliseconds
  } catch (error) {
    alert(`An error occurred: ${error.message}`);
    console.error("Initialization error:", error);
  }
});
</script>

<style scoped>
/* Add any custom styles here */
/* Example styles to enhance the appearance */

#video-container {
  position: relative;
  width: 100%;
  max-width: 640px;
  height: auto;
}

canvas {
  /* Ensure the canvas covers the video element */
  width: 100%;
  height: 100%;
}

.output {
  position: absolute;
  top: 0;
  left: 0;
  color: #fff;
  padding: 10px;
  background: rgba(0, 0, 0, 0.5);
}

/* Responsive Adjustments */
@media (max-width: 768px) {
  #row {
    flex-direction: column;
    padding: 0 2rem;
  }

  #video-container {
    max-width: 100%;
  }
}
</style>

<template>
    <div class="container">
      <!-- Webcam Video Feed -->
      <video
        ref="video"
        class="input_video"
        autoplay
        muted
        playsinline
        width="640"
        height="480"
      ></video>
      
      <!-- Status Display -->
      <div class="status">
        <h2>{{ status }}</h2>
      </div>
      
      <!-- Emoji Displays -->
      <div v-if="showHeart" class="heart">
        ❤️
      </div>
      <div v-if="showHangLoose" class="hang-loose">
        🤙
      </div>
      <div v-if="showWave" class="wave">
        👋
      </div>
    </div>
  </template>
  
  <script>
  import * as tf from '@tensorflow/tfjs';
  
  export default {
    name: 'Text',
    data() {
      return {
        status: 'Initializing...',
        
        // Sign Detection Properties
        sequence: [], // To store keypoints sequence
        sentence: [], // To store detected signs
        threshold: 0.8, // Confidence threshold
        actions: ['hello', 'thanks', 'iloveyou'], // Define your actions
        
        // Emoji Display Flags
        showHeart: false,      // For "I love you" sign
        showHangLoose: false, // For "thank you" sign
        showWave: false,       // For "hello" sign
      };
    },
    methods: {
      /**
       * Processes each video frame by sending it to Mediapipe Holistic.
       */
      async onFrame() {
        if (this.$refs.video && this.$refs.video.readyState === 4) {
          try {
            await this.holistic.send({ image: this.$refs.video });
          } catch (error) {
            console.error('Error sending frame to Holistic:', error);
          }
        }
      },
      
      /**
       * Callback function invoked by Mediapipe Holistic after processing a frame.
       * Handles sign detection without drawing landmarks.
       * @param {Object} results - The results from Mediapipe Holistic.
       */
      onResults(results) {
        try {
          console.log('onResults called');
          
          // Warn if no landmarks are detected
          if (
            !results.poseLandmarks &&
            !results.faceLandmarks &&
            !results.leftHandLandmarks &&
            !results.rightHandLandmarks
          ) {
            console.warn('No landmarks detected in this frame.');
          }
  
          // No drawing logic needed since we're removing landmarks drawing
  
          // Extract keypoints for sign detection
          const keypoints = this.extractKeypoints(results);
  
          // Add keypoints to the sequence
          this.sequence.push(keypoints);
          if (this.sequence.length > 30) {
            this.sequence.shift(); // Maintain only the last 30 frames
          }
  
          // Proceed with prediction if the sequence is complete
          if (this.model && this.sequence.length === 30) {
            const inputTensor = tf.tensor([this.sequence]);
            const prediction = this.model.predict(inputTensor);
            const res = prediction.arraySync()[0];
            const maxIndex = res.indexOf(Math.max(...res));
            const predictedAction = this.actions[maxIndex];
            const confidence = res[maxIndex];
  
            console.log('Predicted action:', predictedAction, 'Confidence:', confidence);
  
            // Check if the prediction meets the confidence threshold
            if (confidence > this.threshold) {
              // Avoid duplicate entries in the sentence
              if (this.sentence.length === 0 || predictedAction !== this.sentence[this.sentence.length - 1]) {
                this.sentence.push(predictedAction);
              }
  
              // Limit the sentence length
              if (this.sentence.length > 5) {
                this.sentence.shift(); // Remove the oldest action
              }
  
              // Update the status display
              this.status = `Action: ${this.sentence.join(' ')}`;
  
              // Handle specific actions
              if (predictedAction === 'iloveyou') {
                this.triggerEmoji('heart');
              } else if (predictedAction === 'thanks') {
                this.triggerEmoji('hangLoose');
              } else if (predictedAction === 'hello') {
                this.triggerEmoji('wave');
              }
            } else {
              // Optionally, handle low-confidence predictions
              // For example, you can reset the sentence or update the status
              // this.sentence = [];
              // this.status = 'No confident action detected.';
            }
          }
        } catch (error) {
          console.error('Error in onResults:', error);
        }
      },
      
      /**
       * Extracts and formats keypoints from Mediapipe results for model prediction.
       * @param {Object} results - The results from Mediapipe Holistic.
       * @returns {Array} - A flat array of keypoints.
       */
      extractKeypoints(results) {
        const pose = results.poseLandmarks || [];
        const face = results.faceLandmarks || [];
        const lh = results.leftHandLandmarks || [];
        const rh = results.rightHandLandmarks || [];
  
        const poseArray = pose.flatMap((res) => [
          res.x,
          res.y,
          res.z,
          res.visibility,
        ]);
        const faceArray = face.flatMap((res) => [res.x, res.y, res.z]);
        const lhArray = lh.flatMap((res) => [res.x, res.y, res.z]);
        const rhArray = rh.flatMap((res) => [res.x, res.y, res.z]);
  
        // Ensure fixed length arrays by padding with zeros if necessary
        const poseData =
          poseArray.length === 33 * 4
            ? poseArray
            : [...poseArray, ...new Array(33 * 4 - poseArray.length).fill(0)];
        const faceData =
          faceArray.length === 468 * 3
            ? faceArray
            : [...faceArray, ...new Array(468 * 3 - faceArray.length).fill(0)];
        const lhData =
          lhArray.length === 21 * 3
            ? lhArray
            : [...lhArray, ...new Array(21 * 3 - lhArray.length).fill(0)];
        const rhData =
          rhArray.length === 21 * 3
            ? rhArray
            : [...rhArray, ...new Array(21 * 3 - rhArray.length).fill(0)];
  
        const keypoints = [...poseData, ...faceData, ...lhData, ...rhData];
  
        return keypoints;
      },
      
      /**
       * Initiates the continuous processing of video frames.
       */
      processVideoFrame() {
        const processFrame = async () => {
          await this.onFrame();
          requestAnimationFrame(processFrame);
        };
        processFrame();
      },
  
      /**
       * Triggers the display of a specific emoji based on the detected action.
       * @param {String} type - The type of emoji to display ('heart', 'hangLoose', 'wave').
       */
      triggerEmoji(type) {
        if (type === 'heart') {
          this.showHeart = true;
          setTimeout(() => {
            this.showHeart = false;
          }, 2000); // Display for 2 seconds
        } else if (type === 'hangLoose') {
          this.showHangLoose = true;
          setTimeout(() => {
            this.showHangLoose = false;
          }, 2000); // Display for 2 seconds
        } else if (type === 'wave') {
          this.showWave = true;
          setTimeout(() => {
            this.showWave = false;
          }, 2000); // Display for 2 seconds
        }
      },
    },
    
    async mounted() {
      if (process.client) {
        try {
          console.log('Starting setup...');
          this.status = 'Loading TensorFlow.js model...';
  
          // Load the TensorFlow.js model
          this.model = await tf.loadLayersModel('/model_web/model.json');
          console.log('Model loaded successfully.');
  
          this.status = 'Loading Mediapipe Holistic...';
  
          // Dynamically import Mediapipe Holistic modules
          const {
            Holistic,
            POSE_CONNECTIONS,
            HAND_CONNECTIONS,
            FACEMESH_TESSELATION,
          } = await import('@mediapipe/holistic');
          const drawingUtils = await import('@mediapipe/drawing_utils');
  
          // Assign imported utilities directly to the instance (not reactive)
          this.POSE_CONNECTIONS = POSE_CONNECTIONS;
          this.HAND_CONNECTIONS = HAND_CONNECTIONS;
          this.FACEMESH_TESSELATION = FACEMESH_TESSELATION;
          this.drawConnectors = drawingUtils.drawConnectors;
          this.drawLandmarks = drawingUtils.drawLandmarks;
  
          // Initialize Mediapipe Holistic
          this.holistic = new Holistic({
            locateFile: (file) => {
              return `https://cdn.jsdelivr.net/npm/@mediapipe/holistic/${file}`;
            },
          });
  
          this.holistic.setOptions({
            modelComplexity: 1,
            smoothLandmarks: true,
            minDetectionConfidence: 0.5,
            minTrackingConfidence: 0.5,
          });
  
          // Bind the onResults callback to ensure correct 'this' context
          this.holistic.onResults(this.onResults);
  
          this.status = 'Accessing webcam...';
  
          // Access the webcam and start processing
          const stream = await navigator.mediaDevices.getUserMedia({ video: true });
          this.$refs.video.srcObject = stream;
          this.$refs.video.play();
  
          // Wait for the video to be ready
          this.$refs.video.onloadedmetadata = () => {
            this.status = 'Processing video...';
            this.processVideoFrame();
          };
        } catch (error) {
          console.error('Error during setup:', error);
          this.status = 'Error initializing application. Check console for details.';
        }
      }
    },
    
    beforeUnmount() {
      // Close Mediapipe Holistic
      if (this.holistic) {
        this.holistic.close();
      }
  
      // Stop all video tracks to release the webcam
      if (this.$refs.video && this.$refs.video.srcObject) {
        const tracks = this.$refs.video.srcObject.getTracks();
        tracks.forEach((track) => track.stop());
      }
    },
  };
  </script>
  
  <style scoped>
  .container {
    position: relative;
    width: 640px;
    height: 480px;
    margin: auto;
    margin-top: 50px;
    border: 2px solid #333;
    border-radius: 10px;
    overflow: hidden;
  }
  
  .input_video {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .status {
    position: absolute;
    bottom: 10px;
    left: 10px;
    background: rgba(0, 0, 0, 0.5);
    padding: 5px 10px;
    border-radius: 5px;
  }
  
  .status h2 {
    color: #fff;
    margin: 0;
    font-size: 16px;
  }
  
  /* Emoji Styles */
  .heart {
    position: absolute;
    bottom: 10px;
    right: 10px;
    font-size: 48px;
    animation: fadeInOut 2s ease-in-out;
  }
  
  .hang-loose {
    position: absolute;
    bottom: 60px; /* Adjust as needed */
    right: 10px;
    font-size: 48px;
    animation: fadeInOut 2s ease-in-out;
  }
  
  .wave {
    position: absolute;
    bottom: 110px; /* Adjust as needed */
    right: 10px;
    font-size: 48px;
    animation: fadeInOut 2s ease-in-out;
  }
  
  @keyframes fadeInOut {
    0% { opacity: 0; transform: translateY(20px); }
    10% { opacity: 1; transform: translateY(0); }
    90% { opacity: 1; transform: translateY(0); }
    100% { opacity: 0; transform: translateY(20px); }
  }
  </style>
  
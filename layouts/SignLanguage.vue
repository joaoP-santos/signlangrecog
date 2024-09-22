<template>
    <div>
      <Webcam ref="webcam" :width="640" :height="480" />
      <div class="output">
        <h1>{{ action }}</h1>
      </div>
    </div>
  </template>
  
  <script>
  import * as tf from '@tensorflow/tfjs';
  import { Holistic } from '@mediapipe/holistic';
  import Webcam from 'vue-web-cam';
  
  export default {
    name: 'SignLanguage',
    components: {
      Webcam,
    },
    data() {
      return {
        action: '',
        model: null,
        actions: ['hello', 'thanks', 'iloveyou'],
        camera: null,
      };
    },
    methods: {
      async onFrame() {
        if (this.$refs.webcam && this.$refs.webcam.video.readyState === 4) {
          await this.holistic.send({ image: this.$refs.webcam.video });
        }
      },
      onResults(results) {
        const keypoints = this.extractKeypoints(results);
        if (this.model && keypoints.length === 1662) {
          const inputTensor = tf.tensor([keypoints]);
          this.model.predict(inputTensor).then((prediction) => {
            const predictedIndex = prediction.argMax(-1).dataSync()[0];
            const action = this.actions[predictedIndex];
            this.action = action;
          });
        }
      },
      extractKeypoints(results) {
        const pose = results.poseLandmarks || [];
        const face = results.faceLandmarks || [];
        const lh = results.leftHandLandmarks || [];
        const rh = results.rightHandLandmarks || [];
  
        const poseArray = pose.flatMap((res) => [res.x, res.y, res.z, res.visibility]);
        const faceArray = face.flatMap((res) => [res.x, res.y, res.z]);
        const lhArray = lh.flatMap((res) => [res.x, res.y, res.z]);
        const rhArray = rh.flatMap((res) => [res.x, res.y, res.z]);
  
        // Ensure fixed length arrays
        const poseData = poseArray.length === 33 * 4 ? poseArray : new Array(33 * 4).fill(0);
        const faceData = faceArray.length === 468 * 3 ? faceArray : new Array(468 * 3).fill(0);
        const lhData = lhArray.length === 21 * 3 ? lhArray : new Array(21 * 3).fill(0);
        const rhData = rhArray.length === 21 * 3 ? rhArray : new Array(21 * 3).fill(0);
  
        const keypoints = [
          ...poseData,
          ...faceData,
          ...lhData,
          ...rhData,
        ];
  
        return keypoints;
      },
    },
    mounted() {
      // Load the model
      tf.loadLayersModel('/model_web/model.json').then((loadedModel) => {
        this.model = loadedModel;
      });
  
      // Set up Mediapipe Holistic
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
  
      this.holistic.onResults(this.onResults);
  
      // Start the camera and process frames
      this.$nextTick(() => {
        if (this.$refs.webcam && this.$refs.webcam.video.readyState === 4) {
          this.camera = new cam.Camera(this.$refs.webcam.video, {
            onFrame: this.onFrame,
            width: 640,
            height: 480,
          });
          this.camera.start();
        }
      });
    },
    beforeDestroy() {
      if (this.camera) {
        this.camera.stop();
      }
    },
  };
  </script>
  
  <style scoped>
  .output {
    position: absolute;
    top: 0;
    left: 0;
    color: #fff;
    padding: 10px;
    background: rgba(0, 0, 0, 0.5);
  }
  </style>
  
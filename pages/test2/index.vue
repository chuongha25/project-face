<template>
  <div>
    <video ref="video" width="640" height="480" autoplay></video>
    <button @click="captureImage">Check for Face</button>
    <canvas
      ref="canvas"
      width="640"
      height="480"
      style="display: none"
    ></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as faceapi from "face-api.js";

const video = ref(null);
const canvas = ref(null);

onMounted(async () => {
  try {
    // Load face-api.js models from the public/models directory
    await faceapi.nets.tinyFaceDetector.loadFromUri("/models");
    await faceapi.nets.faceLandmark68Net.loadFromUri("/models");
    await faceapi.nets.faceRecognitionNet.loadFromUri("/models");

    // Access the camera
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      video.value.srcObject = stream;
    }
  } catch (error) {
    console.error("Error loading models or accessing camera:", error);
  }
});

const captureImage = async () => {
  try {
    const context = canvas.value.getContext("2d");
    context.drawImage(video.value, 0, 0, 640, 480);

    // Perform face detection
    const detections = await faceapi.detectAllFaces(
      canvas.value,
      new faceapi.TinyFaceDetectorOptions()
    );
    if (detections.length > 0) {
      alert("Face detected!");
    } else {
      alert("No face detected.");
    }
  } catch (error) {
    console.error("Error detecting face:", error);
    alert("An error occurred while detecting face.");
  }
};
</script>

<style scoped>
video {
  border: 1px solid black;
}
</style>

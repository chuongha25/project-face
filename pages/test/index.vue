<template>
  <div>
    <h1>Face Detection</h1>
    <video ref="video" autoplay muted width="400" height="400"></video>
    <button @click="startVideo" v-show="!videoStarted">Start Video</button>
    <p v-if="faceDetected">Face Detected</p>
    <p v-else>No Face Detected</p>
    <button @click="detectFaces">Verify Face</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as faceapi from "face-api.js";

const { $geoLocation } = useNuxtApp();

const video = ref(null);
const faceDetected = ref(false);
let detectionInterval = null;
let videoStarted = false;

const startVideo = () => {
  navigator.getUserMedia(
    { video: {} },
    (stream) => {
      video.value.srcObject = stream;
      videoStarted = true;
    },
    (err) => console.error(err)
  );
};

console.log(
  navigator?.geolocation &&
    navigator.geolocation.getCurrentPosition((position) => {
      const latitude = position.coords.latitude;
      const longitude = position.coords.longitude;

      console.log(latitude, longitude);
      // Do something with the position
    })
);

const loadModels = async () => {
  const MODEL_URL =
    "https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js@master/weights";
  await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
};

const detectFaces = async () => {
  try {
    const detections = await faceapi.detectAllFaces(
      video.value,
      new faceapi.TinyFaceDetectorOptions()
    );
    console.log(detections);
    if (detections.length > 0) {
      faceDetected.value = true;
      stopVideo();
      if (detectionInterval) {
        clearInterval(detectionInterval);
      }
    }
  } catch (error) {
    console.log(error);
  }
};

const stopVideo = () => {
  const stream = video.value.srcObject;
  const tracks = stream.getTracks();
  tracks.forEach((track) => track.stop());
  video.value.srcObject = null;
  videoStarted = false;
};

onMounted(async () => {
  await loadModels();
  startVideo();
  detectionInterval = setInterval(detectFaces, 100);
});
</script>

<style scoped>
video {
  display: block;
  margin: auto;
}
</style>

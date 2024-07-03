<template>
  <div class="container mx-auto h-full">
    <h1 class="text-center text-xl font-semibold my-10">
      Face Recognition Check Out
    </h1>
    <div class="w-full">
      <video ref="video" width="420" height="360" autoplay></video>
    </div>
    <div class="w-full flex flex-row justify-evenly mt-10">
      <el-button type="primary" @click="handleCheckIn"
        >Start Check Out</el-button
      >
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as faceapi from "face-api.js";
import { useRouter } from "vue-router";

const video = ref(null);
const isFaceDetected = ref(false);
const position = ref(null);
let faceDetectionInterval = null;
const router = useRouter();

let isVideoActive = false;
let videoStream = null;

const loadModels = async () => {
  await faceapi.nets.tinyFaceDetector.loadFromUri("/models");
  await faceapi.nets.faceLandmark68Net.loadFromUri("/models");
  await faceapi.nets.faceRecognitionNet.loadFromUri("/models");
};

const startVideo = () => {
  if (!isVideoActive) {
    navigator.mediaDevices
      .getUserMedia({ video: {} })
      .then((stream) => {
        videoStream = stream;
        video.value.srcObject = stream;
        isVideoActive = true;
      })
      .catch((err) => {
        console.error("Error accessing video stream: ", err);
      });
  }
};

const stopVideo = () => {
  if (isVideoActive && videoStream) {
    videoStream.getTracks().forEach((track) => track.stop());
    video.value.srcObject = null;
    isVideoActive = false;
    videoStream = null;
  }
};

const detectFace = async () => {
  const detections = await faceapi.detectSingleFace(
    video.value,
    new faceapi.TinyFaceDetectorOptions()
  );
  if (detections) {
    isFaceDetected.value = true;
  } else {
    isFaceDetected.value = false;
  }
};

const handleCheckIn = () => {
  if (faceDetectionInterval) {
    clearInterval(faceDetectionInterval);
  }

  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      async (pos) => {
        position.value = {
          latitude: pos.coords.latitude,
          longitude: pos.coords.longitude,
        };

        await detectFace();

        setTimeout(() => {
          if (isFaceDetected.value) {
            alert("Check-out successful!");
            console.log(
              `Latitude: ${position.value.latitude}, Longitude: ${position.value.longitude}`
            );
            router.push("/");
            stopVideo();
          } else {
            alert("No face detected. Please try again.");
          }
          clearInterval(faceDetectionInterval);
        }, 3000); // Wait 3 seconds to allow face detection
      },
      (err) => {
        alert("Error getting location: " + err.message);
      }
    );
  } else {
    alert("Geolocation is not supported by this browser.");
  }
};

onMounted(async () => {
  await loadModels();
  startVideo();
});
</script>

<style scoped>
video,
canvas {
  display: block;
  margin: 0 auto;
}
</style>

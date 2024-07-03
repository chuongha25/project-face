<template>
  <div class="container mx-auto h-full">
    <h1 class="text-center text-xl font-semibold my-10">
      Face Recognition Check In
    </h1>
    <div class="w-full">
      <video ref="video" width="420" height="360" autoplay></video>
    </div>
    <div class="w-full flex flex-row justify-evenly mt-10">
      <el-button type="primary" @click="handleCheckIn"
        >Start Check In</el-button
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

const getCurrentDate = () => {
  const now = new Date();
  const year = now.getFullYear();
  let month = (now.getMonth() + 1).toString();
  if (month.length === 1) {
    month = "0" + month;
  }
  let day = now.getDate().toString();
  if (day.length === 1) {
    day = "0" + day;
  }
  return `${year}-${month}-${day}`;
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

const saveCheckInDate = () => {
  const lastCheckInDate = localStorage.getItem("lastCheckInDate");
  const currentDate = getCurrentDate();

  if (lastCheckInDate === currentDate) {
    alert("You have already checked in today. Only checkout is allowed.");
    router.push("/");
  } else {
    localStorage.setItem("lastCheckInDate", currentDate);
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
            alert("Check-in successful!");
            console.log(
              `Latitude: ${position.value.latitude}, Longitude: ${position.value.longitude}`
            );
            saveCheckInDate();
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

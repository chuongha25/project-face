<template>
  <div class="register-page">
    <el-card
      style="width: 500px"
      class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"
    >
      <h1 class="text-center text-xl font-semibold my-10">Check Out User</h1>
      <div class="w-full">
        <video ref="video" width="420" height="360" autoplay></video>
      </div>
      <div class="w-full flex flex-row justify-evenly mt-10">
        <el-button type="primary" @click="handleCheckOut">Check Out</el-button>
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import type { UploadUserFile } from "element-plus";
import axios from "axios";
import { ElMessage } from "element-plus";

const video: Ref<HTMLVideoElement | null> = ref(null);

interface Form {
  latitude: number;
  longitude: number;
  file?: UploadUserFile;
  attendance_type: string;
}

const form = reactive<Form>({
  latitude: 16.023606906649366,
  longitude: 108.2260408955065,
  file: undefined,
  attendance_type: "CHECK_OUT",
});

const startVideo = async () => {
  if (navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices
      .getUserMedia({ video: {} })
      .then((stream) => {
        if (video.value) {
          video.value.srcObject = stream;
        }
      })
      .catch((err) => {
        console.error("Error accessing the webcam:", err);
      });
  }
};

const getCurrentLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        form.latitude = position.coords.latitude;
        form.longitude = position.coords.longitude;
        console.log(position);
      },
      (error) => {
        alert("Error getting location: " + error.message);
      }
    );
  } else {
    alert("Geolocation is not supported by this browser.");
  }
};

// getCurrentLocation();

const base64toBlob = (base64Data: string): Blob => {
  const byteString = atob(base64Data.split(",")[1]);
  const mimeString = base64Data.split(",")[0].split(":")[1].split(";")[0];
  const ab = new ArrayBuffer(byteString.length);
  const ia = new Uint8Array(ab);
  for (let i = 0; i < byteString.length; i++) {
    ia[i] = byteString.charCodeAt(i);
  }
  return new Blob([ab], { type: mimeString });
};

const handleCheckOut = async () => {
  // getCurrentLocation();

  await new Promise((resolve) => setTimeout(resolve, 1000));

  if (!video.value) {
    console.error("Video element is not yet available.");
    return;
  }

  const canvas = document.createElement("canvas");
  canvas.width = video.value?.videoWidth ?? 0;
  canvas.height = video.value?.videoHeight ?? 0;
  const context = canvas.getContext("2d");
  if (context && video.value?.srcObject instanceof MediaStream) {
    context.drawImage(video.value, 0, 0, canvas.width, canvas.height);
    const imageData = canvas.toDataURL("image/png");

    // Convert base64 to Blob
    const imageBlob = base64toBlob(imageData);

    try {
      const formData = new FormData();
      formData.append("file", imageBlob, "image.png");
      formData.append("attendance_type", form.attendance_type);

      const res = await axios.post(
        "http://192.168.1.172:8000/api/attendance-managements",
        formData,
        {
          params: {
            latitude: form.latitude,
            longitude: form.longitude,
          },
        }
      );

      const userInfo = JSON.stringify(res.data);
      const now = new Date();
      const endOfDay = new Date(
        now.getFullYear(),
        now.getMonth(),
        now.getDate(),
        23,
        59,
        59,
        999
      );
      const expires = endOfDay.toUTCString();
      document.cookie = `user=${userInfo}; expires=${expires}; path=/`;

      const tracks = video?.value?.srcObject.getTracks();
      tracks.forEach((track) => track.stop());

      ElMessage({
        message: "Congrats, you have successfully checked out!",
        type: "success",
      });

      navigateTo("/");
    } catch (error) {
      console.error("Error calling face recognition API:", error);
    }
  } else {
    console.error("Canvas context or video dimensions are not available.");
  }
};

onMounted(() => {
  getCurrentLocation();
  startVideo();
});
</script>

<style scoped>
.register-page {
  background-image: linear-gradient(to top, #00c6fb 0%, #005bea 100%);
  height: 100vh;
  position: relative;
}
video,
canvas {
  display: block;
  margin: 0 auto;
}
</style>

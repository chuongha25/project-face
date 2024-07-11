<template>
  <div class="register-page">
    <el-card
      style="width: 400px"
      class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"
    >
      <h1 class="text-center text-2xl font-semibold my-10">Register User</h1>
      <el-form
        class="w-full"
        ref="formRef"
        :model="form"
        :rules="rules"
        label-position="top"
        style="max-width: 600px; margin: auto"
      >
        <el-form-item label="Username" prop="user_name">
          <el-input v-model="form.user_name" />
        </el-form-item>
        <el-form-item label="Email" prop="email">
          <el-input v-model="form.email" />
        </el-form-item>
        <el-form-item label="File" prop="file">
          <video ref="video" width="420" height="360" autoplay></video>
        </el-form-item>
        <div class="w-full flex flex-row justify-evenly mt-10">
          <el-button type="primary" @click="onRegister">Register</el-button>
        </div>
      </el-form>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import type { FormInstance, FormRules, UploadUserFile } from "element-plus";
import axios from "axios";
import { ElMessage } from "element-plus";

const video: Ref<HTMLVideoElement | null> = ref(null);

interface Form {
  user_name: string;
  email: string;
  file: string | null;
}

const form = reactive<Form>({
  user_name: "",
  email: "",
  file: null,
});

const formRef = ref<FormInstance>();

const rules = reactive<FormRules<typeof form>>({
  user_name: [
    { required: true, message: "Please input name", trigger: "blur" },
  ],
  email: [{ required: true, message: "Please input email", trigger: "blur" }],
  file: [
    { required: true, message: "Please register your face", trigger: "blur" },
  ],
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

const onRegister = async () => {
  if (!formRef.value) return;

  formRef.value.validate(async (valid) => {
    if (!valid) return;

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

      if (!imageBlob) {
        form.file = null;
        formRef.value?.validateField("file");
        return;
      } else {
        form.file = imageData;
      }

      try {
        const formData = new FormData();
        formData.append("file", imageBlob, "image.png");

        const res = await axios.post(
          "http://192.168.1.172:8000/api/users",
          formData,
          {
            params: {
              user_name: form.user_name,
              email: form.email,
            },
          }
        );

        const tracks = video?.value?.srcObject.getTracks();
        tracks.forEach((track) => track.stop());

        ElMessage({
          message: "Congrats, you have successfully registered!",
          type: "success",
        });
        navigateTo("/");
      } catch (error) {
        console.log(error);
      }
    } else {
      form.file = null;
      formRef.value?.validateField("file");
    }
  });
};

onMounted(startVideo);
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
.upload {
  width: 100%;
}
</style>

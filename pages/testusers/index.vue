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
        <el-form-item label="Upload File" prop="file">
          <el-upload
            class="upload"
            :file-list="fileList"
            action="#"
            :multiple="false"
            :auto-upload="false"
            drag
            v-bind="$attrs"
            :on-change="handleFileUpload"
          ></el-upload>
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

const router = useRouter();

interface Form {
  user_name: string;
  email: string;
  file?: UploadUserFile;
}

const form = reactive<Form>({
  user_name: "",
  email: "",
  file: undefined,
});

const fileList = ref<UploadUserFile[]>();
const formRef = ref<FormInstance>();

const rules = reactive<FormRules<typeof form>>({
  user_name: [
    { required: true, message: "Please input name", trigger: "blur" },
  ],
  email: [{ required: true, message: "Please input email", trigger: "blur" }],
  file: [{ required: true, message: "Please upload file", trigger: "blur" }],
});

const handleFileUpload = (file: UploadUserFile) => {
  form.file = file;
};

const onRegister = async () => {
  if (!formRef.value) return;

  formRef.value.validate(async (valid) => {
    if (!valid) return;

    try {
      const formData = new FormData();

      if (form.file) {
        console.log(form.file);
        formData.append("file", form.file?.raw as File);
      }

      const res = await axios.post(
        "http://192.168.1.183:8000/api/users",
        formData,
        {
          params: {
            user_name: form.user_name,
            email: form.email,
          },
        }
      );

      ElMessage({
        message: "Congrats, you have successfully registered!",
        type: "success",
      });
      navigateTo("/");
    } catch (error) {
      console.log(error);
    }
  });
};
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

<template>
  <div class="register-page">
    <el-card
      style="width: 500px"
      class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"
    >
      <h1 class="text-center text-2xl font-semibold my-10">Register Office</h1>
      <el-form
        class="w-full"
        ref="formRef"
        :model="form"
        :rules="rules"
        label-position="top"
        style="max-width: 600px; margin: auto"
      >
        <el-form-item label="Office Name" prop="office_name">
          <el-input v-model="form.office_name" />
        </el-form-item>
        <el-form-item label="Address" prop="address">
          <el-input v-model="form.address" />
        </el-form-item>
        <el-form-item label="Latitude" prop="latitude">
          <el-input v-model="form.latitude" disabled />
        </el-form-item>
        <el-form-item label="Longitude" prop="longitude">
          <el-input v-model="form.longitude" disabled />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getCurrentLocation"
            >Get Location</el-button
          >
        </el-form-item>
        <div class="flex justify-center mt-8">
          <el-button type="primary" @click="onRegister"
            >Register Office</el-button
          >
        </div>
      </el-form>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { reactive } from "vue";
import { useRouter } from "vue-router";
import type { FormInstance, FormRules } from "element-plus";
import axios from "axios";

const router = useRouter();
const formRef = ref<FormInstance>();

interface Form {
  latitude: number;
  longitude: number;
  office_name: string;
  address: string;
}

const form = reactive<Form>({
  latitude: 0,
  longitude: 0,
  office_name: "",
  address: "",
});

const rules = reactive<FormRules<typeof form>>({
  latitude: [
    { required: true, message: "Please input laitude", trigger: "blur" },
  ],
  longitude: [
    { required: true, message: "Please input longitude", trigger: "blur" },
  ],
  office_name: [
    { required: true, message: "Please input office name", trigger: "blur" },
  ],
  address: [
    { required: true, message: "Please input address", trigger: "blur" },
  ],
});

const getCurrentLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        form.latitude = position.coords.latitude;
        form.longitude = position.coords.longitude;
        console.log("Latitude:", form.latitude, "Longitude:", form.longitude);
      },
      (error) => {
        alert("Error getting location: " + error.message);
      },
      {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0,
      }
    );
  } else {
    alert("Geolocation is not supported by this browser.");
  }
};

const response = ref(null);

const onRegister = async () => {
  if (!formRef.value) return;

  formRef.value.validate(async (vaild) => {
    if (!vaild) return;

    try {
      const res = await axios.post(
        "http://192.168.1.177:8000/api/offices",
        form,
        {
          headers: {
            "Content-Type": "application/json",
            accept: "application/json",
          },
        }
      );
      response.value = res.data;
      router.push("/");
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
</style>

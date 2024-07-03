<template>
  <div class="container mx-auto h-[60vh]">
    <h1 class="text-center text-xl font-semibold mt-10">
      Please select options?
    </h1>
    <div class="w-full flex flex-row justify-evenly md:justify-center mt-10">
      <el-button class="md:mr-2.5" type="primary" @click="navigateToCheckIn"
        >Check In</el-button
      >
      <el-button type="primary" @click="handleCheckOut">Check Out</el-button>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";

const isCheckedIn = ref(false);
const router = useRouter();

const navigateToCheckIn = () => {
  isCheckedIn.value = true;
  router.push("/checkin");
};

const handleCheckOut = () => {
  if (isCheckedIn.value) {
    router.push("/checkout");
  } else {
    alert("Please checkin first.");
  }
};
</script>

<style scoped></style>

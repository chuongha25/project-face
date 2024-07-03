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

const router = useRouter();

const getCurrentDate = () => {
  const now = new Date();
  const year = now.getFullYear();
  let month = (now.getMonth() + 1).toString().padStart(2, "0");
  let day = now.getDate().toString().padStart(2, "0");
  return `${year}-${month}-${day}`;
};

const isCheckedInToday = computed(() => {
  const lastCheckInDate = localStorage.getItem("lastCheckInDate");
  const currentDate = getCurrentDate();
  return lastCheckInDate === currentDate;
});

const navigateToCheckIn = () => {
  router.push("/checkin");
};

const handleCheckOut = () => {
  if (isCheckedInToday.value) {
    router.push("/checkout");
  } else {
    alert("Please check in first.");
  }
};
</script>

<style scoped></style>

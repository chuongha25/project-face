<template>
  <div class="container mx-auto h-full">
    <h1 class="text-center text-xl font-semibold mt-10">
      Check In Successful!
    </h1>
    <div class="w-full flex flex-row justify-evenly mt-10">
      <el-button type="primary" @click="handleCheckout">Check Out</el-button>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";

const router = useRouter();

const handleCheckout = () => {
  router.push("/checkout");
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 60vh;
}
</style>

<template>
  <div class="register-page h-[60vh]">
    <el-card
      style="width: 400px"
      class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"
    >
      <h1 class="text-center text-xl font-semibold mt-7">
        Please Register User and Checkin!
      </h1>
      <div class="w-full mt-10">
        <div class="w-[200px] mx-auto flex flex-col">
          <el-button
            class="mb-7"
            type="primary"
            @click="onCheckIn"
            v-if="!isCheckdIn"
            >Check In</el-button
          >
          <el-button class="no-margin" type="primary" @click="onCheckOut" v-else
            >Check Out</el-button
          >
          <el-button class="no-margin" type="primary" @click="onRegisterUser"
            >Register User</el-button
          >
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import { ElMessage } from "element-plus";

const isCheckdIn = ref(false);

function getCookie(name: string) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) {
    const part = parts.pop();
    if (part !== undefined) {
      return part.split(";").shift() || null;
    }
  }
}

const fetchStatusAttendance = async () => {
  const userCookie = getCookie("user");
  const userInfo = (userCookie && JSON.parse(userCookie)) || null;
  const userId = (userInfo && userInfo.data.user_id) || null;

  if (!userId) {
    isCheckdIn.value = false;
    return;
  }

  try {
    const { data } = await axios.get(
      "http://192.168.1.172:8000/api/attendance-managements",
      {
        params: {
          user_ids: userId,
        },
        headers: {
          Authorization: "Basic YWRtaW46YWRtaW4=",
        },
      }
    );

    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const checkInAttendance = data.data.items.find((item: any) => {
      const createdAt = new Date(item.created_at);
      return (
        item.user_id === userId &&
        item.status === "CHECK_IN" &&
        createdAt >= today
      );
    });
    const checkOutAttendance = data.data.items.find((item: any) => {
      const createdAt = new Date(item.created_at);
      return (
        item.user_id === userId &&
        item.status === "CHECK_OUT" &&
        createdAt >= today
      );
    });

    isCheckdIn.value = !!checkInAttendance && !checkOutAttendance;
  } catch (error) {
    console.error("Error checking check-in status:", error);
  }
};

const onCheckOut = () => {
  navigateTo("/checkout");
};

const onCheckIn = async () => {
  const userCookie = getCookie("user");
  const userInfo = (userCookie && JSON.parse(userCookie)) || null;
  const userId = (userInfo && userInfo.data.user_id) || null;

  if (!userId) {
    navigateTo("/checkin");
  }

  try {
    const { data } = await axios.get(
      "http://192.168.1.172:8000/api/attendance-managements",
      {
        params: {
          user_ids: userId,
        },
        headers: {
          Authorization: "Basic YWRtaW46YWRtaW4=",
        },
      }
    );

    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const checkOutAttendance = data.data.items.find((item: any) => {
      const createdAt = new Date(item.created_at);
      return (
        item.user_id === userId &&
        item.status === "CHECK_OUT" &&
        createdAt >= today
      );
    });

    if (checkOutAttendance) {
      ElMessage({
        message: "You have already checked in today.",
        type: "error",
      });
      return;
    }
  } catch (error) {
    console.log(error);
  }
};

const onRegisterUser = () => {
  navigateTo("/user");
};

onMounted(async () => {
  await fetchStatusAttendance();
});
</script>

<style scoped>
.register-page {
  background-image: linear-gradient(to top, #00c6fb 0%, #005bea 100%);
  height: 100vh;
  position: relative;
}
.no-margin {
  @apply ml-0 mb-7;
}
</style>

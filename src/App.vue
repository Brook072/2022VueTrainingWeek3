<script setup>
import SignInViewVue from "./components/SignInView.vue";
import GoodsListViewVue from "./components/GoodsListView.vue";
import { ref } from "vue";
const loginStatus = ref(false);
function getLoginStatus(isLogined) {
  loginStatus.value = isLogined;
}
</script>

<template>
  <body>
    <main>
      <SignInViewVue
        @sendLoginStatus="getLoginStatus"
        v-if="loginStatus === false"
      ></SignInViewVue>
      <GoodsListViewVue v-else></GoodsListViewVue>
    </main>
  </body>
</template>

<style scoped>
html,
body {
  height: 100%;
  text-align: center;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}
</style>

<script setup>
import axios from "axios";
import { ref } from "vue";
const emits = defineEmits(["sendLoginStatus"]);
const apiUrl = "https://vue3-course-api.hexschool.io";
const username = ref("");
const password = ref("");
const token = localStorage.getItem("token");
const isLogined = ref(false);

function loginConfirmation() {
  if (token === null) {
    return;
  } else {
    axios
      .post(
        `${apiUrl}/v2/api/user/check`,
        {},
        {
          headers: {
            Authorization: token,
          },
        }
      )
      .then((res) => {
        console.log(res);
        if (res.data.success === true) {
          isLogined.value = true;
          emits("sendLoginStatus", isLogined.value);
        }
      })
      .catch((err) => {
        console.log(err);
        return;
      });
  }
}

function loginPost() {
  axios
    .post(`${apiUrl}/v2/admin/signin`, {
      username: username.value,
      password: password.value,
    })
    .then((res) => {
      console.log(res);
      localStorage.setItem("token", res.data.token);
      isLogined.value = true;
      emits("sendLoginStatus", isLogined.value);
    })
    .catch((err) => {
      console.log(err);
    });
}
loginConfirmation();
</script>

<template>
  <div class="container">
    <div class="row justify-content-center">
      <h1 class="h3 mb-3 font-weight-normal text-center">請先登入</h1>
      <div class="col-8">
        <form id="form" class="form-signin">
          <div class="form-floating mb-3">
            <input
              v-model="username"
              type="email"
              class="form-control"
              id="username"
              placeholder="name@example.com"
              autocomplete="username"
              required
              autofocus
            />
            <label for="username">Email address</label>
          </div>
          <div class="form-floating">
            <input
              v-model="password"
              type="password"
              class="form-control"
              id="password"
              placeholder="Password"
              autocomplete="current-password"
              required
            />
            <label for="password">Password</label>
          </div>
          <button
            @click.prevent="loginPost()"
            class="btn btn-lg btn-primary w-100 mt-3"
            type="submit"
          >
            登入
          </button>
        </form>
      </div>
    </div>
    <p class="mt-5 mb-3 text-muted text-center">&copy; 2021~∞ - 六角學院</p>
  </div>
</template>

<style scoped>
.form-signin {
  width: 100%;
  max-width: 330px;
  padding: 15px;
  margin: auto;
}
</style>

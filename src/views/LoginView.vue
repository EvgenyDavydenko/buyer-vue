<template>
  <div class="login">
    <div v-if="step === `enterPhone`">
      <h2>Login</h2>
      <form @submit.prevent="requestLoginCode">
        <label for="phoneNumber">Phone Number: </label>
        <input 
          v-model="phoneNumber" 
          id="phoneNumber" 
          type="text" 
          placeholder="Enter your phone number" 
          required 
        />
        <button type="submit">Get Code</button>
      </form>
    </div>

    <div v-else-if="step === `enterCode`">
      <h2>Enter Code</h2>
      <form @submit.prevent="confirmLogin">
        <label for="logCode">Code:</label>
        <input 
          v-model="logCode" 
          id="logCode" 
          type="text" 
          placeholder="Enter the code" 
          required 
        />
        <button type="submit">Login</button>
      </form>
    </div>

    <div v-else>
      <h2>Welcome, {{ user.lastName }} {{ user.firstName }}</h2>
      <p>Your access token: {{ accessToken }}</p>
    </div>

    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      step: "enterPhone",
      phoneNumber: "",
      logCode: "",
      error: null,
      accessToken: "",
      refreshToken: "",
      user: null,
    };
  },

  methods: {
    async requestLoginCode() {
      this.error = null;

      try {
        const response = await axios.post("http://localhost:80/graphql", {
          query: `
            mutation {
              login(phoneNumber: "${this.phoneNumber}")
            }
          `,
        });

        if (response.data.errors) {
          throw new Error(response.data.errors[0].message);
        }

        console.log("Server response (login):", response.data.data.login); 
        this.logCode = response.data.data.login;
        this.step = "enterCode";
      } catch (error) {
        this.error = error.message;
      }
    },

    async confirmLogin() {
      this.error = null;

      try {
        const response = await axios.post("http://localhost:80/graphql", {
          query: `
            mutation {
              confirmationLogin(phoneNumber: "${this.phoneNumber}", logCode: "${this.logCode}") {
                accessToken
                refreshToken
                user {
                  id
                  firstName
                  lastName
                  phoneNumber
                }
              }
            }
          `,
        });

        if (response.data.errors) {
          throw new Error(response.data.errors[0].message);
        }

        const data = response.data.data.confirmationLogin;
        this.accessToken = data.accessToken;
        this.refreshToken = data.refreshToken;
        this.user = data.user;

        // Сохраняем токен в sessionStorage
        sessionStorage.setItem("accessToken", data.accessToken);
        sessionStorage.setItem("user", JSON.stringify(data.user));
        this.step = "login";
      } catch (error) {
        this.error = error.message;
      }
    },
  },
};
</script>

<style>
.error {
  color: red;
}
</style>

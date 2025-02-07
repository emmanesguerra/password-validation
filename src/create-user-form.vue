<script setup lang="ts">
  import { ref } from 'vue';

  const username = ref('admin');
  const password = ref('ThisPa55wordIsOK');
  const errors = ref<string[]>([]);
  
  const handleSubmit = async () => {
    
    const API_URL = "https://api.challenge.hennge.com/password-validation-challenge-api/001/challenge-signup";
    const TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsiZW1tYW4uZXNndWVycmEyMDEzQGdtYWlsLmNvbSJdLCJpc3MiOiJoZW5uZ2UtYWRtaXNzaW9uLWNoYWxsZW5nZSIsInN1YiI6ImNoYWxsZW5nZSJ9.ycnuZbUwH3R881PV80YXBXlXXdNAuCOVFgDR-orcY2o";

    try {
      const response = await fetch(API_URL, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${TOKEN}`
        },
        body: JSON.stringify({ username: username.value, password: password.value })
      });

      const data = await response.json();
      console.log(data);

      if (!response.ok) {
        errors.value = data.errors || ["bad_request"];
      } else {
        console.log("Success:", data);
      }

    } catch (error) {
      console.error("System Error:", error);
      errors.value = ["bad_request"];
    }
  };

</script>

<template>
  <div class="form-wrapper">
    <form class="form"  @submit.prevent="handleSubmit">
      <!-- make sure the username and password are submitted -->
      <!-- make sure the inputs have the accessible names of their labels -->
      <label for="username">Username</label>
      <input type="text" id="username" v-model="username" required/>

      <label for="password">Password</label>
      <input type="text" id="password" v-model="password" required/>

      <ul v-if="errors.length">
        <li v-for="(error, index) in errors" :key="index" class="error">
          {{ error }}
        </li>
      </ul>

      <button type="submit" class="submit-button">Create User</button>
    </form>
  </div>
</template>

<style scoped>
.form-wrapper {
  max-width: 500px;
  width: 80%;
  background-color: #efeef5;
  padding: 24px;
  margin: auto;
  border-radius: 8px;
}

.form {
  display: flex;
  gap: 8px;
  flex-direction: column;
}

label {
  font-weight: 700;
}

input {
  outline: none;
  padding: 8px 16px;
  height: 40px;
  font-size: 14px;
  background-color: #f8f7fa;
  border: 1px solid rgba(0, 0, 0, 0.12);
  border-radius: 4px;
}

.submit-button {
  outline: none;
  border-radius: 4px;
  border: 1px solid rgba(0, 0, 0, 0.12);
  background-color: #7135d2;
  color: white;
  font-size: 16px;
  font-weight: 500;
  height: 40px;
  padding: 0 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 8px;
  align-self: flex-end;
  cursor: pointer;
}

ul {
  margin-left: 1.5rem;
}
</style>

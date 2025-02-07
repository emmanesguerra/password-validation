<script setup lang="ts">
  import { ref, defineEmits } from 'vue';

  const username = ref('');
  const password = ref('');
  const uerrors = ref<string[]>([]);
  const perrors = ref<string[]>([]);

  const emit = defineEmits(['create-successful']);

  const usernameInput = ref<HTMLInputElement | null>(null);
  const passwordInput = ref<HTMLInputElement | null>(null);

  // Error messages mapping
  const ERROR_MESSAGES: Record<string, string> = {
    "username_empty": "Username is required.",
    "password_empty": "Password is required.",
    "invalid_request": "Invalid request. Missing or incorrect parameters.",
    "too_short": "Password must be at least 10 characters long",
    "too_long": "Password must be at most 24 characters long",
    "no_whitespace": "Password cannot contain spaces",
    "missing_digits": "Password must contain at least one number",
    "missing_uppercase": "Password must contain at least one uppercase letter",
    "missing_lowercase": "Password must contain at least one lowercase letter",
    "not_allowed": "Sorry, the entered password is not allowed, please try a different one.",
    "unauthorized": "Not authenticated to access this resource.",
    "server_error": "Something went wrong, please try again."
  };

  const validatedUsername = () => {
    uerrors.value = [];

    if (!username.value.trim()) {
      uerrors.value.push("username_empty");
    }
  }

  const validatedPassword = () => {
    perrors.value = [];

    if (!password.value.trim()) {
      perrors.value.push("password_empty");
    }
    if (password.value.length < 10) {
      perrors.value.push("too_short");
    }
    if (password.value.length > 24) {
      perrors.value.push("too_long");
    }
    if (/\s/.test(password.value)) {
      perrors.value.push("no_whitespace");
    }
    if (!/\d/.test(password.value)) {
      perrors.value.push("missing_digits");
    }
    if (!/[A-Z]/.test(password.value)) {
      perrors.value.push("missing_uppercase");
    }
    if (!/[a-z]/.test(password.value)) {
      perrors.value.push("missing_lowercase");
    }
  }

  const validateForm = (): boolean => {
    validatedUsername();
    validatedPassword();

    // adding auto focus for invalid field
    focusFirstError();

    return uerrors.value.length === 0 && perrors.value.length === 0;
  };

  const focusFirstError = () => {
    if (uerrors.value.length > 0 && usernameInput.value) {
      usernameInput.value.focus();
    } else if (perrors.value.length > 0 && passwordInput.value) {
      passwordInput.value.focus();
    }
  };
  
  const handleSubmit = async () => {

    if (!validateForm()) {
      return;
    }
    
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

      // Check if the response is JSON
      if (response.headers.get('Content-Type')?.includes('application/json')) {
          const data = await response.json();
          if (!response.ok) {
            perrors.value = data.errors || ["invalid_request"];
          } else {
            emit('create-successful');
          }
      } else {
        // text/plain
        const status = response.status;
        let errorKey = "unauthorized";
        if (status === 500) {
          errorKey = "server_error";
        }

        throw new Error(ERROR_MESSAGES[errorKey]);
      }
    } catch (error) {
      alert(error.message);
      formReset();
    }
  };

  const formReset = () => {
    username.value = '';
    password.value = '';
    uerrors.value = [];
    perrors.value = [];
  }

</script>

<template>
  <div class="form-wrapper">
    <form class="form"  @submit.prevent="handleSubmit">
      <!-- make sure the username and password are submitted -->
      <!-- make sure the inputs have the accessible names of their labels -->
      <label for="username">Username</label>
      <input 
        type="text" 
        id="username" 
        v-model="username" 
        ref="usernameInput"
        aria-required="true"
        :aria-invalid="uerrors.length > 0 ? 'true' : 'false'"
        aria-describedby="username-error"
        @blur="validatedUsername"
        />

      <div id="username-error" v-if="uerrors.length" role="alert" aria-live="assertive" aria-atomic="true">
        <ul>
          <li v-for="(error, index) in uerrors" :key="index">{{ ERROR_MESSAGES[error] || "Unknown error occurred" }}</li>
        </ul>
      </div>

      <label for="password">Password</label>
      <input 
        type="password" 
        id="password" 
        v-model="password"
        ref="passwordInput"
        :aria-invalid="perrors.length > 0 ? 'true' : 'false'"
        aria-describedby="password-error"
        @blur="validatedPassword"
        />

      <div id="password-error" v-if="perrors.length" role="alert" aria-live="assertive" aria-atomic="true">
        <ul>
          <li v-for="(error, index) in perrors" :key="index">{{ ERROR_MESSAGES[error] || "Unknown error occurred" }}</li>
        </ul>
      </div>

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
  font-size: 90%;
  color: #f00;
}

input[aria-invalid="true"] {
  border: 1px solid #f00;
}
</style>

<template>
  <div class="signup">
    <h1>Sign up</h1>
    <ul>
      <li v-for="error in errors" v-bind:key="error">{{ error }}</li>
    </ul>
    <p>Name:</p>
    <input type="text" v-model="newUserParams.name" />
    <p>Email:</p>
    <input type="text" v-model="newUserParams.email" />
    <p>Password:</p>
    <input type="password" v-model="newUserParams.password" />
    <p>Password confirmation:</p>
    <input type="password" v-model="newUserParams.password_confirmation" />
    <p></p>
    <button v-on:click="submit()">Sign up</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data: function () {
    return {
      newUserParams: {},
      errors: [],
    };
  },
  methods: {
    submit: function () {
      axios
        .post("/users", this.newUserParams)
        .then((response) => {
          console.log(response.data);
          this.$router.push("/login");
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
        });
    },
  },
};
</script>

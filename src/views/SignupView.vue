<template>
  <div class="signup">
    <h1>Sign up</h1>
    <p>Name:</p>
    <input type="text" />
    <p>Email:</p>
    <input type="text" />
    <p>Password:</p>
    <input type="text" />
    <p>Password confirmation:</p>
    <input type="text" />
    <p></p>
    <button>Sign up</button>
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

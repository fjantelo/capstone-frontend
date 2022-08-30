<template>
  <nav v-if="!isLoggedIn">
    <router-link to="/">Home</router-link>
    |
    <router-link to="/signup">Sign up</router-link>
    |
    <router-link to="/login">Log in</router-link>
  </nav>
  <nav v-if="isLoggedIn">
    <router-link to="/">Home</router-link>
    |
    <router-link to="/logout">Log out</router-link>
  </nav>
  <router-view />
</template>

<script>
export default {
  data: function () {
    return {
      isLoggedIn: false,
    };
  },
  watch: {
    $route: function () {
      this.isLoggedIn = !!localStorage.jwt;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}
</style>

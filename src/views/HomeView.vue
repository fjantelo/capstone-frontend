<template>
  <div class="home">
    <div v-if="!isLoggedIn">
      <h1>CollabList</h1>
      <p>Create Music Playlists together with friends.</p>
      <p>To get started, Sign up or Log in.</p>
    </div>
    <div v-if="isLoggedIn">
      <p>Welcome, {{ user }}</p>
      <h1>My Playlists</h1>
      <div v-for="playlist in playlists" v-bind:key="playlist.id">
        <h3>{{ playlist.name }}</h3>
        <p>{{ playlist.description }}</p>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import axios from "axios";

export default {
  data: function () {
    return {
      playlists: [],
      user: "",
      isLoggedIn: false,
    };
  },
  created: function () {
    this.isLoggedIn = !!localStorage.jwt;
    this.user = localStorage.getItem("userName");
    this.indexPlaylists();
  },
  methods: {
    indexPlaylists: function () {
      axios.get("/playlists.json").then((response) => {
        this.playlists = response.data;
        console.log("User's playlists:", this.playlists);
      });
    },
  },
};
</script>

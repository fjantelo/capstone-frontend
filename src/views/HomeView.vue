<template>
  <div class="home">
    <div>
      <h1>CollabList</h1>
      <p>Create Music Playlists together with friends.</p>
      <p>To get started, Sign up or Log in.</p>
    </div>
    <div>
      <p>Welcome, {{ user }}</p>
      <button v-on:click="newPlaylist()">New Playlist</button>
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
    };
  },
  created: function () {
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
    newPlaylist: function () {
      this.$router.push("/playlists/new");
    },
  },
};
</script>

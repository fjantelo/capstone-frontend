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
        <router-link :to="`/playlists/${playlist.id}`">
          <h3>{{ playlist.name }}</h3>
        </router-link>
        <p>{{ playlist.description }}</p>
        <button v-on:click="leavePlaylist(playlist)">Leave Playlist</button>
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
      userId: 0,
      isLoggedIn: false,
    };
  },
  created: function () {
    this.isLoggedIn = !!localStorage.jwt;
    this.user = localStorage.getItem("userName");
    this.userId = localStorage.getItem("userId");
    this.indexPlaylists();
  },
  methods: {
    indexPlaylists: function () {
      axios.get("/playlists.json").then((response) => {
        this.playlists = response.data;
        console.log("User's playlists:", this.playlists);
      });
    },
    leavePlaylist: function (playlist) {
      console.log(this.userId);
      console.log(playlist.id);
      axios.delete(`/user_playlists?user_id=${this.userId}&playlist_id=${playlist.id}`).then((response) => {
        console.log("Success!", response.data);
        this.$router.go();
      });
    },
  },
};
</script>

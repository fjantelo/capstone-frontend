<template>
  <div class="playlists-show">
    <h1>{{ playlist.name }}</h1>
    <h4>{{ playlist.description }}</h4>
    <p v-for="song in songs" v-bind:key="song.id">{{ song.title }} by {{ song.artist }}</p>
    <h4>Users in this Playlist</h4>
    <p v-for="user in users" v-bind:key="user.id">{{ user.name }}</p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data: function () {
    return {
      playlist: {},
      songs: [],
      users: [],
    };
  },
  created: function () {
    axios.get("/playlists/" + this.$route.params.id + ".json").then((response) => {
      this.playlist = response.data;
      this.songs = response.data.songs;
      this.users = response.data.users;
    });
  },
};
</script>

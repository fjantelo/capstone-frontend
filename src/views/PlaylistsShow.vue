<template>
  <div class="playlists-show">
    <h1>{{ playlist.name }}</h1>
    <h4>{{ playlist.description }}</h4>
    <form v-on:submit.prevent="searchSongs(searchQuery)">
      <label>Search for a song to add:</label>
      <input type="text" v-model="searchQuery" />
      <input type="submit" value="Search" />
    </form>
    <div v-for="song in songs" v-bind:key="song.id">
      <p>{{ song.title }}</p>
    </div>
    <h4>Users in this Playlist</h4>
    <p v-for="user in users" v-bind:key="user.id">{{ user.name }}</p>
    <button v-on:click="addUserModal()">Add User</button>
  </div>
  <dialog id="add-user">
    <form method="dialog">
      <h3>Add a User</h3>
      <p>
        Enter user's email:
        <input type="text" v-model="newUserEmail" />
      </p>
      <div>
        <button v-on:click="addUser()">Submit</button>
        <button>Cancel</button>
      </div>
    </form>
  </dialog>
  <dialog id="add-song">
    <form method="dialog">
      <div v-for="result in searchResults" v-bind:key="result">
        <h3>{{ result.snippet.title }}</h3>
        <p>{{ result.snippet.description }}</p>
        <p>Channel: {{ result.snippet.channelTitle }}</p>
        <button v-on:click="createSong(result)">Add to playlist</button>
      </div>
      <p></p>
      <button>Cancel</button>
    </form>
  </dialog>
</template>

<script>
import axios from "axios";

export default {
  data: function () {
    return {
      playlist: {},
      songs: [],
      users: [],
      newUserEmail: "",
      searchQuery: "",
      searchResults: [],
    };
  },
  methods: {
    addUserModal: function () {
      this.newUserEmail = "";
      document.querySelector("#add-user").showModal();
    },
    addSongModal: function () {
      this.searchQuery = "";
      document.querySelector("#add-song").showModal();
    },
    addUser: function () {
      axios
        .post("/user_playlists.json", { email: this.newUserEmail, playlist_id: this.$route.params.id })
        .then((response) => {
          console.log("Success!", response.data);
          this.users.push(response.data[1]);
        })
        .catch((error) => console.log(error.response));
    },
    createSong: function (song) {
      axios
        .post("/songs.json", {
          url: song.id.videoId,
          channel: song.snippet.channelTitle,
          title: song.snippet.title,
          thumbnail: song.snippet.thumbnails.high.url,
          playlist_id: this.$route.params.id,
        })
        .then((response) => {
          console.log(response.data);
        });
    },
    searchSongs: function (query) {
      this.searchResults = [];
      axios.get(`/songs/search?query=${query}`).then((response) => {
        this.searchResults = response.data.items;
        console.log(this.searchResults);
      });
      this.addSongModal();
    },
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

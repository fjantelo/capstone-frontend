<template>
  <div class="playlists-show">
    <h1>{{ playlist.name }}</h1>
    <h4>{{ playlist.description }}</h4>
    <p v-for="song in songs" v-bind:key="song.id">{{ song.title }} by {{ song.artist }}</p>
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
    };
  },
  methods: {
    addUserModal: function () {
      this.newUserEmail = "";
      document.querySelector("#add-user").showModal();
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

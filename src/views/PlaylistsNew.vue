<template>
  <div class="playlists-new">
    <form v-on:submit.prevent="createPlaylist()">
      <h1>New Playlist</h1>
      <label>Name:</label>
      <p></p>
      <input type="text" v-model="newPlaylistParams.name" />
      <p></p>
      <label>Description:</label>
      <p></p>
      <input type="text" v-model="newPlaylistParams.description" />
      <p></p>
      <input type="submit" value="Submit" />
    </form>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data: function () {
    return {
      newPlaylistParams: {},
      errors: [],
      status: "",
    };
  },
  methods: {
    createPlaylist: function () {
      axios
        .post("/playlists.json", this.newPlaylistParams)
        .then((response) => {
          console.log(response.data);
          this.$router.push("/");
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
          this.status = error.response.status;
        });
    },
  },
};
</script>

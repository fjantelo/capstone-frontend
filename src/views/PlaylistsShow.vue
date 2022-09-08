<template>
  <div class="playlists-show">
    <p>Welcome, {{ userName }}</p>
    <h1>{{ playlist.name }}</h1>
    <h4>{{ playlist.description }}</h4>
    <form v-on:submit.prevent="searchSongs(searchQuery)">
      <label>Search for a song to add:</label>
      <input type="text" v-model="searchQuery" />
      <input type="submit" value="Search" />
      <p></p>
    </form>
    <div v-for="song in songs" v-bind:key="song.url">
      <p>
        {{ song.title }}
        <!-- Song Modal -->
        <button v-on:click="playSongModal(song.url)">Play Video</button>
        <!-- Use MusicPlayer component -->
        <button v-on:click="playPauseButtonHandler(song)">Play</button>
        <button v-on:click="removeSong(song)">Remove</button>
      </p>
    </div>
    <h4>Users in this Playlist</h4>
    <div v-for="user in users" v-bind:key="user.id">
      <p v-if="user.id != userId">
        {{ user.name }}
        <button v-on:click="removeUser(user)">Remove</button>
      </p>
    </div>
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
        <iframe
          width="560"
          height="315"
          :src="`https://www.youtube.com/embed/${result.id.videoId}`"
          title="YouTube video player"
          frameborder="0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
        <button v-on:click="createSong(result)">Add to playlist</button>
      </div>
      <p></p>
      <button>Cancel</button>
    </form>
  </dialog>
  <dialog id="play-song">
    <form method="dialog">
      <iframe
        width="560"
        height="315"
        :src="`https://www.youtube.com/embed/${songURL}`"
        title="YouTube video player"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen
      ></iframe>
      <p></p>
      <button>Exit</button>
    </form>
  </dialog>
</template>

<script>
import axios from "axios";

export default {
  emits: ["changeSong", "changePlayerState"],
  props: { currentSong: Object, playerState: Boolean },
  data: function () {
    return {
      userName: "",
      userId: 0,
      playlist: {},
      songs: [],
      users: [],
      newUserEmail: "",
      searchQuery: "",
      searchResults: [],
      songURL: "",
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
    playSongModal: function (url) {
      // load song, then open modal with the appropriate link
      this.songURL = url;
      document.querySelector("#play-song").showModal();
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
          this.$router.go();
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
    removeUser: function (user) {
      axios.delete(`/user_playlists?user_id=${user.id}&playlist_id=${this.$route.params.id}`).then((response) => {
        console.log("Success!", response.data);
        this.$router.go();
      });
    },
    removeSong: function (song) {
      axios.delete(`/songs/${song.id}`).then((response) => {
        console.log("Success!", response.data);
        this.$router.go();
      });
    },
    playPauseButtonHandler(song) {
      // check if the song on which this button was clicked is the current song
      // Yes => check playerState
      // playing => pause
      // paused => play
      // No => change song
      console.log("song.url: ", song.url);
      console.log("this.currentSong.url: ", this.currentSong.url);
      if (song.url === this.currentSong.url) {
        this.$emit("changePlayerState", !this.playerState);
      } else {
        // Determine which songs are following the current song
        let upNext = this.songs.filter((s) => this.songs.indexOf(s) > this.songs.indexOf(song));
        // console.log(upNext.map((s) => s.title));
        console.log(song);
        console.log(upNext);
        this.$emit("changeSong", song, upNext);
      }
    },
    changeSongLocal: function (song) {
      // Determine which songs are following the current song
      let upNext = this.songs.filter((s) => this.songs.indexOf(s) > this.songs.indexOf(song));
      // console.log(upNext.map((s) => s.title));
      this.$emit("changeSong", song, upNext);
    },
  },
  created: function () {
    this.userName = localStorage.getItem("userName");
    this.userId = localStorage.getItem("userId");
    axios.get("/playlists/" + this.$route.params.id + ".json").then((response) => {
      this.playlist = response.data;
      this.songs = response.data.songs;
      this.users = response.data.users;
    });
  },
};
</script>

<style>
.play-btn,
.pause-btn,
.add-btn,
.added-btn {
  fill: var(--font-high);
}
.play-btn:hover,
.pause-btn:hover,
.add-btn:hover,
.added-btn:hover {
  cursor: pointer;
}
/* .play-btn .p1 {
  d: path("M6.79 5.093A.5.5 0 0 0 6 5.5v5a.5.5 0 0 0 .79.407l3.5-2.5a.5.5 0 0 0 0-.814l-3.5-2.5z");
}
.play-btn .p2 {
  d: path(
    "M0 4a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2V4zm15 0a1 1 0 0 0-1-1H2a1 1 0 0 0-1 1v8a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V4z"
  );
}
.play-btn:hover path {
  d: path(
    "M0 12V4a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2zm6.79-6.907A.5.5 0 0 0 6 5.5v5a.5.5 0 0 0 .79.407l3.5-2.5a.5.5 0 0 0 0-.814l-3.5-2.5z"
  );
}
.pause-btn .p1 {
  d: path(
    "M6.25 5C5.56 5 5 5.56 5 6.25v3.5a1.25 1.25 0 1 0 2.5 0v-3.5C7.5 5.56 6.94 5 6.25 5zm3.5 0c-.69 0-1.25.56-1.25 1.25v3.5a1.25 1.25 0 1 0 2.5 0v-3.5C11 5.56 10.44 5 9.75 5z"
  );
}
.pause-btn .p2 {
  d: path(
    "M0 4a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2V4zm15 0a1 1 0 0 0-1-1H2a1 1 0 0 0-1 1v8a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V4z"
  );
}
.pause-btn:hover path {
  d: path(
    "M0 12V4a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2zm6.25-7C5.56 5 5 5.56 5 6.25v3.5a1.25 1.25 0 1 0 2.5 0v-3.5C7.5 5.56 6.94 5 6.25 5zm3.5 0c-.69 0-1.25.56-1.25 1.25v3.5a1.25 1.25 0 1 0 2.5 0v-3.5C11 5.56 10.44 5 9.75 5z"
  );
} */
</style>

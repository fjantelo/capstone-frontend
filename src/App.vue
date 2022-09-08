<template>
  <nav v-if="!isLoggedIn">
    <router-link to="/">Home</router-link>
    |
    <router-link to="/signup">Sign up</router-link>
    |
    <router-link to="/login">Log in</router-link>
  </nav>
  <nav v-if="isLoggedIn">
    <router-link to="/">My Playlists</router-link>
    |
    <router-link to="/playlists/new">New Playlist</router-link>
    |
    <router-link to="/logout">Log out</router-link>
  </nav>
  <router-view
    @change-song="passChangeSong"
    :current-song="currentSong"
    @change-player-state="playPause"
    :player-state="playerState"
  />
  <MusicPlayer
    class="fixed-bottom"
    ref="MusicPlayer"
    @change-song="updateCurrentSong"
    @change-player-state="updatePlayerState"
    :current-song="currentSong"
    :player-state="playerState"
  />
</template>

<script>
import MusicPlayer from "@/components/MusicPlayer.vue";

export default {
  components: {
    MusicPlayer,
  },
  data: function () {
    return {
      isLoggedIn: false,
      currentSong: {}, // sets title in MusicPlayer
      playerState: false,
    };
  },
  watch: {
    $route: function () {
      this.isLoggedIn = !!localStorage.jwt;
    },
  },
  methods: {
    // This method is called when one of the other views emits the changeSong event. The argument is listening for the payload emitted, so even though the @change-song="passChangeSong" does not contain an argument, it is required here.
    playPause() {
      this.$refs.musicPlayer.playPause();
    },
    updatePlayerState() {
      this.playerState = !this.playerState;
      console.log("from App.vue: state is playing: ", this.playerState);
    },
    updateCurrentSong(song) {
      this.currentSong = song;
    },
    passChangeSong(song, upNext) {
      console.log("A song was selected and passed to the /App page. Passing along to the MusicPlayer: ", song);
      console.log(upNext);
      this.currentSong = song;
      this.$refs.MusicPlayer.selectedSongController(song, upNext);
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

@media (max-width: 768px) {
  .form-main {
    width: 90%;
  }
}
@media (min-width: 768px) {
  .form-main {
    width: 50%;
  }
}

.playing {
  /* background-color: var(--color1-5); */
  color: var(--color1-5);
}
</style>

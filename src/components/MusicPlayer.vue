<template>
  <div id="player">
    <div class="container">
      <!-- Queue / UpNext -->
      <div class="collapse" id="collapseUpNext">
        <div class="card p-2">
          <div v-if="upNext.length == 0">
            <i>No songs in queue</i>
          </div>
          <div v-else>
            <div v-for="song in upNext" :key="song.url">
              <div class="card song m-2">
                <button class="button-no-format song" @click="changeSong(song)">
                  {{ song.title }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <!-- Navigation buttons -->
        <div class="col-auto my-1 p-1 d-flex align-items-center">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            viewBox="0 0 16 16"
            class="clickable"
            @click="prevSong()"
          >
            <path
              d="M4 4a.5.5 0 0 1 1 0v3.248l6.267-3.636c.54-.313 1.232.066 1.232.696v7.384c0 .63-.692 1.01-1.232.697L5 8.753V12a.5.5 0 0 1-1 0V4z"
            />
          </svg>
        </div>
        <div class="col-auto my-1 p-1 d-flex align-items-center">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            viewBox="0 0 16 16"
            class="clickable"
            @click="playPause()"
          >
            <path d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z" />
            <path :d="playPauseGraphic" />
          </svg>
        </div>
        <div class="col-auto my-1 p-1 d-flex align-items-center">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            viewBox="0 0 16 16"
            class="clickable"
            @click="nextSong()"
          >
            <path
              d="M12.5 4a.5.5 0 0 0-1 0v3.248L5.233 3.612C4.693 3.3 4 3.678 4 4.308v7.384c0 .63.692 1.01 1.233.697L11.5 8.753V12a.5.5 0 0 0 1 0V4z"
            />
          </svg>
          <div id="youtube-player"></div>
        </div>

        <!-- About and scroll -->
        <div class="col m-1">
          <i v-if="currentSong.title" v-html="currentSong.title"></i>
          <i v-else>No song selected</i>
          <div class="slidecontainer">
            <span>{{ this.formattedCurrentTime }}</span>
            <input
              type="range"
              min="0"
              :max="this.duration"
              :value="currentTime"
              class="slider"
              id="progressBar"
              @change="scrollTo()"
            />
            <span>{{ formatTime(this.duration) }}</span>
          </div>
        </div>
        <div class="col-auto m-1">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            class="clickable"
            viewBox="0 0 16 16"
            data-bs-toggle="collapse"
            data-bs-target="#collapseUpNext"
            aria-expanded="false"
            aria-controls="collapseUpNext"
          >
            <path d="M12 13c0 1.105-1.12 2-2.5 2S7 14.105 7 13s1.12-2 2.5-2 2.5.895 2.5 2z" />
            <path fill-rule="evenodd" d="M12 3v10h-1V3h1z" />
            <path d="M11 2.82a1 1 0 0 1 .804-.98l3-.6A1 1 0 0 1 16 2.22V4l-5 1V2.82z" />
            <path
              fill-rule="evenodd"
              d="M0 11.5a.5.5 0 0 1 .5-.5H4a.5.5 0 0 1 0 1H.5a.5.5 0 0 1-.5-.5zm0-4A.5.5 0 0 1 .5 7H8a.5.5 0 0 1 0 1H.5a.5.5 0 0 1-.5-.5zm0-4A.5.5 0 0 1 .5 3H8a.5.5 0 0 1 0 1H.5a.5.5 0 0 1-.5-.5z"
            />
          </svg>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* global YT */
// import YT from "https://www.youtube.com/iframe_api";
export default {
  name: "MusicPlayer",
  emits: ["changeSong", "changePlayerState"],
  props: {
    currentSong: Object,
  },
  data: function () {
    return {
      YTplayer: {},
      playerIsReady: false,
      // Data used in progress bar
      duration: 0,
      currentTime: 0,
      formattedCurrentTime: 0,
      formattedDuration: 0,
      progressBar: 0,
      // Play/pause controls
      // Set player state for other views
      playerState: false, // true => playing, false => paused
      // Set to the play button by default
      playPauseGraphic:
        "M6.271 5.055a.5.5 0 0 1 .52.038l3.5 2.5a.5.5 0 0 1 0 .814l-3.5 2.5A.5.5 0 0 1 6 10.5v-5a.5.5 0 0 1 .271-.445z",
      path_play:
        "M6.271 5.055a.5.5 0 0 1 .52.038l3.5 2.5a.5.5 0 0 1 0 .814l-3.5 2.5A.5.5 0 0 1 6 10.5v-5a.5.5 0 0 1 .271-.445z",
      path_pause:
        "M5 6.25a1.25 1.25 0 1 1 2.5 0v3.5a1.25 1.25 0 1 1-2.5 0v-3.5zm3.5 0a1.25 1.25 0 1 1 2.5 0v3.5a1.25 1.25 0 1 1-2.5 0v-3.5z",
      upNext: {},
    };
  },
  methods: {
    // Creates the youtube player from which the music will come
    onYouTubeIframeAPIReady(song) {
      // console.log("called onYouTubeIframeAPIReady");
      this.YTplayer = new YT.Player("youtube-player", {
        height: "0",
        width: "0",
        videoId: song.url,
        playerVars: { autoplay: 1 },
        events: {
          onReady: () => {
            this.YTplayer.setPlaybackQuality("small");
            this.playerIsReady = true;
            // console.log("Music player is now ready: ", this.playerIsReady);
          },
          onStateChange: this.musicController,
        },
      });
    },
    // Routes actions in the music player whenever the YT player state changes
    musicController() {
      // console.log(this.YTplayer.getPlayerState());
      this.duration = this.YTplayer.getDuration();
      this.formattedDuration = this.formatTime(this.duration);
      this.playPauseGraphicChange();
      this.YTplayer.getPlayerState() !== 1 ? this.stopProgressBar() : this.startProgressBar();
      this.$emit("changePlayerState");
    },
    // Creates and destroys the 1s interval that updates the scroll bar
    startProgressBar() {
      this.progressBar = setInterval(this.trackTime, 1000);
    },
    stopProgressBar() {
      clearInterval(this.progressBar);
    },
    formatTime(input_seconds) {
      // Formatting for render on screen
      var dateObj = new Date(Math.floor(input_seconds * 1000));
      var hours = dateObj.getUTCHours();
      var minutes = dateObj.getUTCMinutes();
      var seconds = dateObj.getSeconds();
      var o1 = input_seconds > 3600 ? hours.toString().padStart(2, "0") + ":" : "";
      var o2 = minutes.toString().padStart(2, "0") + ":" + seconds.toString().padStart(2, "0");
      return o1 + o2;
    },
    trackTime() {
      this.currentTime = this.YTplayer.getCurrentTime();
      this.formattedCurrentTime = this.formatTime(this.currentTime);
      if (this.duration - this.currentTime < 1) {
        this.nextSong();
      }
    },
    scrollTo() {
      this.YTplayer.seekTo(document.getElementById("progressBar").value, true);
      this.currentTime = document.getElementById("progressBar").value;
      this.formattedCurrentTime = this.formatTime(document.getElementById("progressBar").value);
    },
    selectedSongController(song, upNext) {
      console.log(upNext);
      // check if the player is already created
      console.log("Selected song received in AudioComponent: ", song);
      if (this.playerIsReady) {
        // console.log("Player is already created, so passing the song to changeSong method");
        this.changeSong(song);
      } else {
        // console.log("Music player is not yet created, so calling the onYoutubeIframeReady method");
        this.onYouTubeIframeAPIReady(song);
      }
      this.upNext = upNext;
    },
    changeSong(song) {
      console.log("changing song to " + song);
      this.YTplayer.loadVideoById(song.url);
      this.$emit("changeSong", song);
      this.upNext = this.upNext.filter((s) => this.upNext.indexOf(s) > this.upNext.indexOf(song));
    },
    nextSong() {
      console.log("current upNext is: ", this.upNext);
      if (this.upNext.length > 0) {
        let next = this.upNext.shift();
        this.changeSong(next);
        // this.$emit("changeSong", next);
      }
    },
    playPause() {
      if (
        this.YTplayer.getPlayerState() === YT.PlayerState.PLAYING ||
        this.YTplayer.getPlayerState() === YT.PlayerState.BUFFERING
      ) {
        this.YTplayer.pauseVideo();
      } else {
        this.YTplayer.playVideo();
      }
    },
    playPauseGraphicChange() {
      this.playPauseGraphic = this.YTplayer.getPlayerState() === 1 ? this.path_pause : this.path_play;
    },
  },
};
</script>

<style>
#player {
  background-color: var(--neutral-2);
}
.clickable {
  cursor: pointer;
  cursor: hand;
  fill: var(--font-high);
}
.clickable:hover {
  fill: var(--color1-5);
}
/* *********************** */
/* Range slider formatting */
/* *********************** */
/* Styles needed to override basic appearance for range slider */
input[type="range"] {
  -webkit-appearance: none; /* Hides the slider so that custom slider can be made */
  width: 100%; /* Specific width is required for Firefox. */
  background: transparent; /* Otherwise white in Chrome */
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
}
input[type="range"]:focus {
  outline: none; /* Removes the blue border. You should probably do some kind of focus styling for accessibility reasons though. */
}
input[type="range"]::-ms-track {
  width: 100%;
  cursor: pointer;
  /* Hides the slider so custom styles can be added */
  background: transparent;
  border-color: transparent;
  color: transparent;
}
/*Chrome*/
@media screen and (-webkit-min-device-pixel-ratio: 0) {
  input[type="range"] {
    overflow: hidden;
    width: 80%;
    -webkit-appearance: none;
    background-color: var(--neutral-3);
    border-radius: 5px;
  }
  input[type="range"]::-webkit-slider-runnable-track {
    height: 10px;
    -webkit-appearance: none;
    color: var(--neutral-3);
    margin-top: 0px; /* margin-top = (track height in pixels / 2) - (thumb height in pixels /2) */
  }
  input[type="range"]::-webkit-slider-thumb {
    width: 10px;
    -webkit-appearance: none;
    height: 10px;
    border-radius: 50%;
    cursor: ew-resize;
    background: var(--color1-5);
    box-shadow: calc(-80vw - 5px) 0 0 80vw var(--color1-4);
  }
}
</style>

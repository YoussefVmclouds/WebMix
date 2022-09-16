<template>
  <!-- Janus Video -->
  <div id="camsContianer">
    <span :key="i" v-for="(camera, i) in cameras">
      <p @click="playAudio(`liveCam${i}`, `liveAud${i}`)">{{ camera }}</p>
      <video
        :id="`liveCam${i}`"
        class="live-video"
        playsinline
        autoplay
        muted
        @click="$emit('main-camera', camera, i)"
      ></video>
      <audio
        :id="`liveAud${i}`"
        class="live-audio"
        playsinline
        muted
        autoplay
      ></audio>
      <span>
        <!-- audio bars canvas for auido feedbacak is on - off? -->
        <!-- <canvas :id="`audioBarsCanvas${i}`" width="190" height="60"></canvas> -->
        <div
          :id="`audioBarsCanvas${i}`"
          width="200"
          height="60"
          style="margin: 0px 10px"
        ></div>
      </span>
    </span>
    <!-- for user interaction on page to allow audio bars to work -->
    <v-btn
      v-if="!startFeed"
      v-bind="startFeed"
      color="#ff0000"
      @click="loadFeed"
    >
      PUSH ME!</v-btn
    >
  </div>
</template>

<script>
// import Janus from "./janus";
import { Janus } from "janus-gateway";

export default {
  name: "LiveCam",
  props: {
    janus: {
      type: Object,
    },
  },
  data() {
    return {
      streaming: [],
      cameras: [],
      tempCameras: [],
      serverList: [],
      watchID: null,
      startFeed: false,
      camCount: null,
    };
  },
  created() {
    // temp conition to mimic url fetching cams from server
    if (this.$route.params.eventKey == "123-test-123") {
      // this.watchID = "Opus"
      this.watchID = " ";
    }
    // production version should be something like this
    else {
      this.watchID = this.$route.params.eventKey;
    }
  },
  mounted() {
    // auto check each X time with server if live cams updated
    window.setInterval(() => {
      this.checkServer();
    }, 2500);
  },
  methods: {
    // check for new live cams on server to show them later on the Mixer page
    checkServer() {
      this.initJanus();
      if (this.cameras.length == this.camCount) {
        // console.log("Same Cams Count!", this.camCount);
        return;
      } else {
        this.camCount = this.cameras.length;
        console.log("Cams Count Changed to: ", this.camCount);
        if (this.startFeed) {
          this.playCameras();
        }
      }
    },
    // start video feed if we have ready live streams from server
    loadFeed() {
      this.startFeed = true;
      this.initJanus();
    },
    // Init Janus, get server list of streams, according to url Key aka "this.watchID"
    // checks streams ID beofre adding them to (camaras) to avoid repeating sources
    initJanus() {
      this.janus.attach({
        opaqueId: "test",
        plugin: "janus.plugin.streaming",
        success: (pluginHandle) => {
          if (pluginHandle) {
            let body = { request: "list" };
            pluginHandle.send({
              message: body,
              success: (result) => {
                this.serverList = result.list;
                // console.log(this.serverList)
                if (this.startFeed) {
                  this.serverList.forEach((element) => {
                    if (this.watchID != null) {
                      if (element.description.search(this.watchID) > -1) {
                        if (this.cameras.indexOf(element.id) === -1) {
                          this.cameras.push(element.id);
                          this.tempCameras.push({
                            id: element.id,
                            play: false,
                          });
                        }
                      }
                    }
                  });
                }
              },
            });
          }
        },
      });
    },
    // playing live streams via (tempCameras) avoid problems with auto play policy
    playCameras() {
      for (let i = 0; i < this.tempCameras.length; i++) {
        if (this.tempCameras[i].play == false) {
          this.janus.attach({
            opaqueId: "test-" + i,
            plugin: "janus.plugin.streaming",
            detached: () => {
              console.log("cleaning!");
            },
            success: (pluginHandle) => {
              if (pluginHandle) {
                this.streaming.push({ id: i, plugin: pluginHandle });
                let body = { request: "watch", id: this.tempCameras[i].id };
                pluginHandle.send({ message: body });
              }
            },
            error: (error) => {
              console.log(error);
            },
            onmessage: (msg, jsep) => {
              if (jsep !== undefined && jsep !== null) {
                const foundStream = this.streaming.find((s) => s.id === i);
                if (jsep.type === "offer") {
                  foundStream.plugin.createAnswer({
                    jsep,
                    media: { audioSend: false, videoSend: false },
                    success: function (jsep) {
                      const body = { request: "start" };
                      foundStream.plugin.send({ message: body, jsep: jsep });
                    },
                    error: function (error) {
                      Janus.error("WebRTC error:", error);
                    },
                  });
                }
              }
            },
            onremotetrack: (track, mid, on) => {
              console.log(track);
              var streamV = null;
              var streamA = null;
              if (track.kind === "audio") {
                streamA = new MediaStream([track]);
              } else if (track.kind === "video") {
                streamV = new MediaStream([track]);
              } else {
                Janus.error("Server replied with unexpected track!");
              }
              const element = document.getElementById(`liveCam${i}`);

              if (streamV) {
                Janus.attachMediaStream(element, streamV);
              }
              if (streamA) {
                const elementAudio = document.getElementById(`liveAud${i}`);
                Janus.attachMediaStream(elementAudio, streamA);
                elementAudio.muted = true;
                // liveAud${i}
                // const elementAudioCanvas = document.getElementById(`audioBarsCanvas${i}`);
                this.visualize(i, streamA, 90);
              }
              this.tempCameras[i].play = true;
            },
            // onremotestream: (stream) => {
            //   const element = document.getElementById(`liveCam${i}`);
            //   Janus.attachMediaStream(element, stream);
            //   const elementAudio = document.getElementById(
            //     `audioBarsCanvas${i}`
            //   );
            //   Janus.attachMediaStream(elementAudio, stream);
            //   this.visualize(i, stream, 90);
            //   this.tempCameras[i].play = true;
            // },
          });
        }
      }
    },
    // play audio of prv box on clicking text above the small video box
    playAudio(live, audio) {
      if (document.getElementById(audio).muted == false) {
        document
          .getElementById(live)
          .parentNode.firstChild.classList.remove("orange-Box");
        document.getElementById(audio).muted = true;
      } else {
        document
          .getElementById(live)
          .parentNode.firstChild.classList.add("orange-Box");
        document.getElementById(audio).volume = 1;
        document.getElementById(audio).muted = false;
      }
    },
    // trying to make audio bars here for audio feedback per live source
    visualize(id, mediaStream, frameHeight) {
      // The number of bars that should be displayed
      const NBR_OF_BARS = 10;
      // console.log(frameHeight);
      // Create an audio context
      const ctx = new AudioContext();

      // Create an audio source
      const audioSource = ctx.createMediaStreamSource(mediaStream);

      // Create an audio analyzer
      const analayzer = ctx.createAnalyser();
      analayzer.fftSize = 1024;

      // Connect the source, to the analyzer, and then back the the context's destination
      audioSource.connect(analayzer);
      // audioSource.connect(ctx.destination);

      // Print the analyze frequencies
      const frequencyData = new Uint8Array(analayzer.frequencyBinCount);
      analayzer.getByteFrequencyData(frequencyData);

      // Get the visualizer container
      const visualizerContainer = document.getElementById(
        `audioBarsCanvas${id}`
      );

      // Create a set of pre-defined bars
      for (let i = 0; i < NBR_OF_BARS; i++) {
        const bar = document.createElement("DIV");
        bar.setAttribute("id", `bar${id}` + i);
        bar.setAttribute("class", "visualizer-container__bar");
        visualizerContainer.appendChild(bar);
      }

      // This function has the task to adjust the bar heights according to the frequency data
      function renderFrame() {
        // Update our frequency data array with the latest frequency data
        analayzer.getByteFrequencyData(frequencyData);

        for (let i = 0; i < NBR_OF_BARS; i++) {
          // Since the frequency data array is 1024 in length, we don't want to fetch
          // the first NBR_OF_BARS of values, but try and grab frequencies over the whole spectrum
          const index = i * 40;
          // fd is a frequency value between 0 and 255
          // const fd = frequencyData[index] / 4;
          const fd = frequencyData[index];
          fd = (((frameHeight * 95) / 100) * fd) / 255;

          // Fetch the bar DIV element
          const bar = document.querySelector(`#bar${id}` + i);
          if (!bar) {
            continue;
          }

          // If fd is undefined, default to 0, then make sure fd is at least 4
          // This will make make a quiet frequency at least 4px high for visual effects
          const barHeight = Math.max(1 / 4, fd || 0);
          bar.style.height = barHeight + "px";
          if (barHeight >= 0 && barHeight < (frameHeight * 5) / 100) {
            bar.style.background = `linear-gradient(
                180deg,
                rgba(0, 0, 0, 1) 0%,
                rgba(69, 182, 75, 1) 100%
              )`;
          } else if (
            barHeight >= (frameHeight * 5) / 100 &&
            barHeight < (frameHeight * 60) / 100
          ) {
            bar.style.background = `linear-gradient(
                180deg,
                rgba(0, 0, 0, 1) 0%,
                rgba(69, 182, 75, 1) 2%,
                rgba(106, 255, 0, 1) 100%
                )`;
          } else if (
            barHeight >= (frameHeight * 60) / 100 &&
            barHeight < (frameHeight * 85) / 100
          ) {
            bar.style.background = `linear-gradient(
                180deg,
                rgba(0, 0, 0, 1) 0%,
                rgba(69, 182, 75, 1) 2%,
                rgba(106, 255, 0, 1) 65%,
                rgba(255, 254, 0, 1) 100%
              )`;
          } else {
            bar.style.background = `linear-gradient(
                180deg,
                rgba(0, 0, 0, 1) 0%,
                rgba(69, 182, 75, 1) 2%,
                rgba(106, 255, 0, 1) 65%,
                rgba(255, 254, 0, 1) 85%,
                rgba(255, 0, 0, 1)  100%
              )`;
          }
        }

        // At the next animation frame, call ourselves
        window.requestAnimationFrame(renderFrame);
      }

      renderFrame();
    },
  },
};
</script>

<style>
body,
textarea,
input,
select {
  background: 0;
  border-radius: 0;
  font: 16px sans-serif;
  margin: 0;
}
.addon,
.btn-sm,
.nav,
textarea,
input,
select {
  outline: 0;
  font-size: 14px;
}
.smooth {
  transition: all 0.2s;
}
.btn,
.nav a {
  text-decoration: none;
}
.container {
  margin: 0 20px;
  width: auto;
}
@media (min-width: 1310px) {
  .container {
    margin: auto;
    width: 1270px;
  }
}
.btn,
h2 {
  font-size: 2em;
}
.msg {
  background: #def;
  border-left: 5px solid #59d;
  padding: 1.5em;
}
.warning {
  background: #fdd;
  border-left: 5px solid #e44;
}
#errorMsg {
  display: none;
}
#meterGraph {
  height: 30px;
  width: 400px;
  border: 1px solid #000;
  margin: 10px 10px 10px 10px;
}
#meter {
  background: #00ff00;
  width: 0%;
  height: 100%;
}
canvas {
  border: 1px solid #666;
  background: #def;
}
.visualizer-container__bar {
  display: inline-block;
  background: rgb(0, 0, 0);
  background: linear-gradient(
    180deg,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 255, 1, 0.4976584383753502) 8%,
    rgba(37, 255, 0, 1) 100%
  );
  margin: 0 2px;
  width: 16px;
}
.orange-Box {
  z-index: 100;
  /* outline-offset: -2rem; */
  color: rgb(255, 132, 0) !important;
  /* border-bottom: 3rem solid rgba(255, 132, 0, 0.948) !important; */
  /* border-left: 5rem solid rgba(255, 0, 0, 0.805) !important; */
  /* border-right: 5rem solid rgba(255, 0, 0, 0.805) !important; */
}
</style>

<style scoped lang="sass">
.live-video
  width: 200px
  height: 115px
  background: black
  border: 3px solid #666
  margin: 5px 10px
</style>
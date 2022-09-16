<template>
  <!-- Janus Video -->
  <div>
    <div>
      <div
        class="mt-5 text-center"
        v-if="!loaded"
        style="
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
        "
      >
        <v-progress-circular
          :size="200"
          color="secondary"
          indeterminate
        ></v-progress-circular>
      </div>
      <video
        :id="`janusVideoPGM`"
        class="janus-video"
        playsinline
        autoplay
        width="100%"
        height="100%"
        muted
        @click="playAudio(`janusVideoPGM`, `janusAudioPGM`)"
      ></video>
      <audio
        :id="`janusAudioPGM`"
        class="live-audio"
        playsinline
        autoplay
      ></audio>
    </div>
  </div>
</template>

<script>
// import Janus from "./janus";
import { Janus } from "janus-gateway";

export default {
  name: "JanusVideo",
  props: {
    // cameras: {
    //   type: Array,
    //   required: true
    // },
    janus: {
      type: Object,
    },
  },
  data() {
    return {
      streaming: [],
      cameraPGM: [],
      serverList: [],
      watchID: null,
      loaded: false,
    };
  },
  created() {
    // temp conition to mimic url fetching cams from server
    if (this.$route.params.eventKey == "123-test-123") {
      this.watchID = "Qaf";
    } else {
      this.watchID = `PGM${this.$route.params.eventKey}PGM"`;
    }
  },
  mounted() {
    this.initJanus();
  },
  methods: {
    initJanus() {
      this.janus.attach({
        opaqueId: "PGM",
        plugin: "janus.plugin.streaming",
        success: (pluginHandle) => {
          if (pluginHandle) {
            let body = { request: "list" };
            // this.watchID = this.$route.query.pgm
            pluginHandle.send({
              message: body,
              success: (result) => {
                this.serverList = result.list;
                this.serverList.forEach((element) => {
                  if (this.watchID != null) {
                    if (element.description.search(this.watchID) > -1) {
                      this.cameraPGM.push(element.id);
                      this.requestInfo();
                      // this.playPGM();
                    }
                  }
                });
              },
            });
          }
        },
      });
    },
    requestInfo() {
      this.janus.attach({
        opaqueId: "PGM",
        plugin: "janus.plugin.streaming",
        success: (pluginHandle) => {
          if (pluginHandle) {
            let body = { request: "info", id: this.cameraPGM[0] };
            // this.watchID = this.$route.query.pgm
            pluginHandle.send({
              message: body,
              success: (result) => {
                console.log(result);
                this.playPGM();
              },
            });
          }
        },
      });
    },
    playPGM() {
      this.janus.attach({
        opaqueId: "PGM",
        plugin: "janus.plugin.streaming",
        success: (pluginHandle) => {
          if (pluginHandle) {
            this.streaming.push({
              id: this.cameraPGM[0],
              plugin: pluginHandle,
            });
            let body = { request: "watch", id: this.cameraPGM[0] };
            pluginHandle.send({ message: body });
          }
        },
        error: (error) => {
          console.log(error);
        },
        onmessage: (msg, jsep) => {
          if (jsep !== undefined && jsep !== null) {
            const foundStream = this.streaming.find(
              (s) => s.id === this.cameraPGM[0]
            );
            if (jsep.type === "offer") {
              foundStream.plugin.createAnswer({
                jsep: jsep,
                tracks: [{ type: "data" }],
                success: (jsep) => {
                  var body = { request: "start" };
                  foundStream.plugin.send({
                    message: body,
                    jsep: jsep,
                  });
                },
                error: (error) => {
                  Janus.error("WebRTC error:", error);
                },
              });
            }
          }
        },
        onremotetrack: (track, mid, on) => {
          console.log(track, mid, on);
          var streamV;
          var streamA;
          console.log(
            "audSrc:",
            document.getElementById(`janusAudioPGM`).srcObject
          );
          console.log(
            "vidSrc:",
            document.getElementById(`janusVideoPGM`).srcObject
          );

          if (
            track.kind === "audio"
            // && document.getElementById(`janusAudioPGM`).srcObject == null
          ) {
            streamA = new MediaStream([track]);

            const elementAudio = document.getElementById(`janusAudioPGM`);
            Janus.attachMediaStream(elementAudio, streamA);

            elementAudio.muted = true;
          } else if (track.kind === "video") {
            streamV = new MediaStream([track]);
            const element = document.getElementById(`janusVideoPGM`);

            Janus.attachMediaStream(element, streamV);
          } else {
            Janus.error("Server replied with unexpected track!");
          }

          setTimeout(() => {
            this.loaded = true;
          }, 5000);
        },
      });
    },
    playAudio(liveVideo, liveAudio) {
      if (document.getElementById(liveAudio).muted == false) {
        document.getElementById(liveVideo).classList.remove("red-Box");
        document.getElementById(liveAudio).muted = true;
      } else {
        document.getElementById(liveVideo).classList.add("red-Box");
        document.getElementById(liveAudio).volume = 1;
        document.getElementById(liveAudio).muted = false;
      }
    },
  },
};
</script>

<style>
.red-Box {
  z-index: 100;
  /* outline-offset: -2rem; */
  outline: 5px rgba(255, 2, 2, 0.825) solid;
  /* border-bottom: 1rem solid rgba(255, 0, 0, 0.805) !important; */
  /* border-left: 5rem solid rgba(255, 0, 0, 0.805) !important; */
  /* border-right: 5rem solid rgba(255, 0, 0, 0.805) !important; */
}
</style>

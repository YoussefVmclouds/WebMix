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
        @click="playAudio(`janusVideoPGM`)"
      ></video>
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
    requestID:{
      type: Number,
    },
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
      isCalled: false,
    };
  },
  created() {
    // temp conition to mimic url fetching cams from server
    if(this.$route.params.eventKey == "123-test-123"){
        this.watchID = "Multistream"  
      }
    },
  mounted () {
    // this.initJanus()
    this.cameraPGM.push(this.requestID)
    // console.log("pgm ID is: ", this.cameraPGM)
    this.playPGM()
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
                      this.playPGM();
                    }
                  }
                });
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
                jsep,
                media: { audioSend: false, videoSend: false },
                success: (jsep) => {
                  const body = { request: "start" };
                  foundStream.plugin.send({ message: body, jsep: jsep });
                },
                error: (error) => {
                  Janus.error("WebRTC error:", error);
                },
              });
            }
          }
        },
        onremotestream: (stream) => {
          if (!this.isCalled) {
            console.log(stream, stream.getVideoTracks, stream.getAudioTracks);
            const element = document.getElementById(`janusVideoPGM`);
            Janus.attachMediaStream(element, stream);
            this.isCalled = true;
            setTimeout(() => {
              this.loaded = true;
            }, 5000);
          }
        },
      });
    },
    playAudio(liveVideo) {
      if (document.getElementById(liveVideo).muted == false) {
        document.getElementById(liveVideo).classList.remove("red-Box");
        document.getElementById(liveVideo).muted = true;
      } else {
        document.getElementById(liveVideo).classList.add("red-Box");
        document.getElementById(liveVideo).volume = 1;
        document.getElementById(liveVideo).muted = false;
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

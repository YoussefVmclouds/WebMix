<template>
  <!-- Live Cam Video Feeds -->
  <div>
    <span
    :key="i"
      v-for="(camera, i) in cameras">
    <h3>{{camera}}</h3>
    <video
      :id="`janusVideo${i}`"
      class="janus-video"
      playsinline
      autoplay
      controls
      muted
    >
</video>
    </span>
  </div>  
</template>

<script>
import Janus from './janus'

export default {
  name: 'JanusVideo',
  props: {
    // cameras: {
    //   type: Array,
    //   required: true
    // },
    janus: {
      type: Object
    }
  },
  data () {
    return {
      streaming: [],
      cameras: [],
      tempCameras:[],
      serverList:[],
      watchID:null,
      startFeed: false,
    }
  },
  mounted () {
    //   first call to play any ready cams from server
    this.initJanus();
    // auto check each X time with server if live cams changed
    window.setInterval(() => {
      this.checkServer();
    }, 1000);
    // mimic adding new cams on server after each Y time
    window.setInterval(() => {
      this.pushCam();
    }, 7777);  
    },
  methods: {
    //   check for new live cams on server to show them later on the Mixer page
    checkServer() {
      if (this.cameras.length == this.camCount) {
        console.log("Same Cams Count!", this.camCount);
        return;
      } else {
        this.camCount = this.cameras.length;
        console.log("Cams Count Changed to: ", this.camCount);
        this.playCameras();
      }
    },
    // usless function, just to mimic adding new cameras like it is new on server side
    pushCam() {
      if (this.cameras.length == 0) {
        this.cameras.push(1);
        return;
      }
      if (this.cameras.length == 1) {
        this.cameras.push(123);
        return;
      }
      if (this.cameras.length == 2) {
        this.cameras.push(3);
        return;
      }
      if (this.cameras.length == 3) {
        this.cameras.push(2);
        return;
      }
    },
    loadFeed(){
      this.startFeed= true
      // this.initJanus()
    },
    // Init Janus, get server list of streams, according to url Key
    initJanus () {
      this.janus.attach({
        opaqueId: "test",
        plugin: "janus.plugin.streaming",
        success: (pluginHandle) => {
            if (pluginHandle) {
            let body = { 'request': 'list' };
            this.watchID = this.$route.query.prv;
            pluginHandle.send({
              'message': body,
                success: (result) => {
                this.serverList = result.list;
                console.log(this.serverList)
                this.serverList.forEach((element) => {
                  if (this.watchID != null) {
                    if (element.description.search(this.watchID) > -1) {
                      // temp comment till we test our Janus server
                      // this.cameras.push(element.id)
                    }
                  }
                })
                    },
                })
            }
        },
        });
        this.playCameras()
        },
    playCameras(){
      for (let i = 0; i < this.cameras.length; i++) {
        this.janus.attach(
          {
            opaqueId: 'test-' + i,
            plugin: 'janus.plugin.streaming',
            detached: () => {
              console.log("cleaning!")
            },
        success: (pluginHandle) => {
              if (pluginHandle) {
                this.streaming.push({ id: i, plugin: pluginHandle })
                let body = { 'request' : 'watch', 'id': this.cameras[i] }
                pluginHandle.send({ 'message': body })
            }
        },
        error: (error) => { console.log(error) },
        onmessage: (msg, jsep) => {
            if (jsep !== undefined && jsep !== null) {
                const foundStream = this.streaming.find(s => s.id === i)
                if (jsep.type === 'offer') {
                    foundStream.plugin.createAnswer(
                    {
                      jsep,
                      media: { audioSend: false, videoSend: false },
                      success: function (jsep) {
                        const body = { 'request': 'start' }
                        foundStream.plugin.send({ 'message': body, 'jsep': jsep })
                      },
                      error: function (error) {
                        Janus.error('WebRTC error:', error)
                      }
                    }
                  )
                }
              }
            },
        onremotestream: (stream) => {
              const element = document.getElementById(`janusVideo${i}`)
              Janus.attachMediaStream(element, stream)
            },
            
        })
      }
      },
        // end test
  }
}  
</script>

<style scoped lang="sass">
.janus-video
  width: 200px
  height: 115px
  background: black
  border: 3px solid #666
  margin: 5px 10px
</style>
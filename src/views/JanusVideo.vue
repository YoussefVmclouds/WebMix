<template>
  <!-- Janus Video -->
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
      @click="$emit('view-video',camera)"
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
      camCount:null,
    }
  },
  created() {
    // temp conition to mimic url fetching cams from server
    if(this.$route.params.eventKey == "123-test-123"){
        // this.watchID = "Opus"  
        this.watchID = " "  
      }
    },
  mounted () {
    // first call to prepare any ready cams from server
    this.initJanus();
    // auto check each X time with server if live cams updated
    window.setInterval(() => {
      this.checkServer();
    }, 2500); 
    },
  methods: {
     // check for new live cams on server to show them later on the Mixer page
    checkServer() {
      this.initJanus()
      if (this.cameras.length == this.camCount) {
        console.log("Same Cams Count!", this.camCount);
        return;
      } else {
        this.camCount = this.cameras.length;
        console.log("Cams Count Changed to: ", this.camCount);
        this.playCameras();
      }
    },
    loadFeed(){
      this.startFeed= true
      // this.initJanus()
    },
    // Init Janus, get server list of streams, according to url Key
    // checks streams ID beofre adding them to (camaras) to avoid repeating sources
    initJanus () {
      this.janus.attach({
        opaqueId: "test",
        plugin: "janus.plugin.streaming",
        success: (pluginHandle) => {
          if (pluginHandle) {
            let body = { 'request': 'list' };
            pluginHandle.send({
              'message': body,
              success: (result) => {
                this.serverList = result.list;
                // console.log(this.serverList)
                  this.serverList.forEach((element) => {
                    if (this.watchID != null) {
                      if (element.description.search(this.watchID) > -1) {
                          if(this.cameras.indexOf(element.id) === -1){
                            this.cameras.push(element.id)
                            this.tempCameras.push({id:element.id,play:false})
                          } 
                      }
                    }
                  })
                },
              })
            }
          },
        });
      },
    // playing live streams via (tempCameras) avoid problems with auto play policy 
    playCameras(){
      for (let i = 0; i < this.tempCameras.length; i++) {
        if(this.tempCameras[i].play == false){
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
                let body = { 'request' : 'watch', 'id': this.tempCameras[i].id }
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
              this.tempCameras[i].play=true
            },
            
        })
        }
      }
      },
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
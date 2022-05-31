<template>
  <!-- Janus Video -->
  <div id="camsContianer">
    <span
    :key="i"
      v-for="(camera, i) in cameras">
    <h3>{{camera}}</h3>
    <video
      :id="`liveCam${i}`"
      class="live-video"
      playsinline
      autoplay
      muted
      @click="$emit('main-camera',camera,i)"
    >
</video>
<span>
    <canvas :id="`audioBarsCanvas${i}`" width="190" height="60"></canvas>
</span>
    </span>
    <v-btn v-if="!startFeed" v-bind="startFeed" color="#ff0000" @click="loadFeed"> PUSH ME!</v-btn>
  </div>  
</template>

<script>
import Janus from './janus'

export default {
  name: 'LiveCam',
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
    // this.initJanus();
    
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
      this.initJanus()
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
                if (this.startFeed) {
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
                }
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
              const element = document.getElementById(`liveCam${i}`)
              Janus.attachMediaStream(element, stream)
              const elementAudio = document.getElementById(`audioBarsCanvas${i}`)
              Janus.attachMediaStream(elementAudio, stream)
              this.visualize(i,stream)
              this.tempCameras[i].play=true
            },
            
        })
        }
      }
      },

    visualize(id,mediaStream ) {
      let canvasElement = document.getElementById(`audioBarsCanvas${id}`);
      let canvasContext = canvasElement.getContext("2d");
      let audioContext = new AudioContext();
      // console.log(audioContext.state)
      audioContext.resume()
      // console.log(audioContext.state)
      let analyserNode = audioContext.createAnalyser();
      let bufferLength = analyserNode.frequencyBinCount;
      let dataArray = new Uint8Array(bufferLength);
      const mediaStreamSource = audioContext.createMediaStreamSource(mediaStream);
      mediaStreamSource.connect(analyserNode);
      draw();
      function draw() {
        requestAnimationFrame(draw);
        analyserNode.getByteTimeDomainData(dataArray);

        canvasContext.clearRect(0, 0, canvasElement.width, canvasElement.height);
        canvasContext.lineWidth = 1.25;
        canvasContext.strokeStyle = "#33cc33";
        canvasContext.beginPath();
        canvasContext.moveTo(0, canvasElement.height / 2);

        let x = 0;
        let sliceWidth = canvasElement.width / bufferLength;
        for (let i = 0; i < bufferLength; i++) {
          let v = dataArray[i] / 128;
          let y = canvasElement.height / 2 * v;
          canvasContext.lineTo(x, y);
          x += sliceWidth;
        }

        canvasContext.lineTo(canvasElement.width, canvasElement.height / 2);
        canvasContext.stroke();

      }
    },
  }
} 
  
</script>

<style>
body,textarea,input,select {
  background:0;
  border-radius:0;
  font:16px sans-serif;
  margin:0
}
.addon,.btn-sm,.nav,textarea,input,select{
  outline:0;
  font-size:14px
}
.smooth{
    transition:all .2s
}
.btn,.nav a{
    text-decoration:none
}
.container{
    margin:0 20px;
    width:auto
}
@media(min-width:1310px){
  .container{
    margin:auto;
    width:1270px
  }
}
.btn,h2{
  font-size:2em
}
.msg{
  background:#def;
  border-left:5px solid #59d;
  padding:1.5em
}
.warning{
  background:#fdd;
  border-left:5px solid #e44
}
#errorMsg{
  display:none
}
#meterGraph{
  height:30px;
  width:400px;
  border:2px solid rgb(4, 255, 0);
  margin:10px 10px 10px 10px;
}
#meter{
  background: #00ff00;
  width: 0%;
  height: 100%;
}
canvas {
  border: 1px solid #666;
  background: rgb(255, 255, 255);
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
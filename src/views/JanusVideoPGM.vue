<template>
  <!-- Janus Video -->
  <div>
    <div>
    <video
      :id="`janusVideoPGM`"
      class="janus-video"
      playsinline
      autoplay
      width= 100%
      height= auto
      controls
      muted
    ></video>
    </div>
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
      cameraPGM: [],
      serverList:[],
      watchID:null,
    }
  },
    created() {
    // temp conition to mimic url fetching cams from server
    if(this.$route.params.eventKey == "123-test-123"){
        this.watchID = "Multistream"  
      }
    },
  mounted () {
    this.initJanus()
  },
  methods: {
    initJanus () {
      this.janus.attach(
        {
        opaqueId: 'PGM',
        plugin: 'janus.plugin.streaming',
        success:  (pluginHandle) =>{
          if (pluginHandle) {
            let body = { 'request': 'list' }
            // this.watchID = this.$route.query.pgm
            pluginHandle.send({ 'message': body,
            success: (result)=> {
              this.serverList=result.list
              this.serverList.forEach(element => {
                if(this.watchID !=null){
                  if(element.description.search(this.watchID)>-1){
                    this.cameraPGM.push(element.id)
                    this.playPGM()
                    }
                  }
                });
              }
            })
              }
            }
          })
    },

    playPGM(){
      this.janus.attach(
        {
        opaqueId: 'PGM',
        plugin: 'janus.plugin.streaming',
        success: (pluginHandle) => {
          if (pluginHandle) {
            this.streaming.push({ id: this.cameraPGM[0], plugin: pluginHandle })
            let body = { 'request': 'watch', id: this.cameraPGM[0] }
            pluginHandle.send({ 'message': body })
            }
          },
        error: (error) => { console.log(error) },
        onmessage: (msg, jsep) => {
          if (jsep !== undefined && jsep !== null) {
            const foundStream = this.streaming.find(s => s.id === this.cameraPGM[0])
            if (jsep.type === 'offer') {
              foundStream.plugin.createAnswer(
              {
              jsep,
              media: { audioSend: false, videoSend: false },
              success: (jsep) => {
                const body = { 'request': 'start' }
                foundStream.plugin.send({ 'message': body, 'jsep': jsep })
                },
              error: (error) => {
                Janus.error('WebRTC error:', error)
                }
              })
              }
            }
          },
        onremotestream: (stream) => {
          const element = document.getElementById(`janusVideoPGM`)
          Janus.attachMediaStream(element, stream)
          },
      })
    }
  }
}
</script>

// <style scoped lang="sass">
// .janus-video
//   min-width: 100% !important
//   // width: atuo
//   min-height: 100% !important
//   // height: auto
//   object-fit: fill
//   background: black
//   display: block
// </style>
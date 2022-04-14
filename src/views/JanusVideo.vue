<template>
  <!-- Janus Video -->
  <div>
    <span
    :key="camera"
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
      serverList:[],
      watchID:null,
      startFeed: false,
    }
  },
  mounted () {
      this.initJanus()
  },
  methods: {
    loadFeed(){
      this.startFeed= true
      // this.initJanus()
    },
    // Init Janus
    initJanus () {
        //  this.janus.attach.bind({x:1})
         this.janus.attach(
          {
            opaqueId: 'test-',

            plugin: 'janus.plugin.streaming',
            success:  (pluginHandle) =>{
              if (pluginHandle) {
                let body = { 'request': 'list' }
                this.watchID = this.$route.query.prv
                console.log("looking for (",this.watchID,") as cameras preview")
                pluginHandle.send({ 'message': body,
                  success: (result)=> {
                    console.log(result)
                    this.serverList=result.list
                    this.serverList.forEach(element => {
                    if(this.watchID !=null){
                      if(element.description.search(this.watchID)>-1){
                        this.cameras.push(element.id)
                      }
                      }
                    });
                    // test
  const vm = this
for (let i = 0; i < vm.cameras.length; i++) {
        this.janus.attach(
          {
            opaqueId: 'test-' + i,
            plugin: 'janus.plugin.streaming',
            success: function (pluginHandle) {
              if (pluginHandle) {
                vm.streaming.push({ id: i, plugin: pluginHandle })
                let body = { 'request': 'watch', id: vm.cameras[i] }
                pluginHandle.send({ 'message': body })
              }
            },
            error: function (error) { console.log(error) },
            onmessage: function (msg, jsep) {
              if (jsep !== undefined && jsep !== null) {
                const foundStream = vm.streaming.find(s => s.id === i)
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
            onremotestream: function (stream) {
            //   console.log(`iteration ${i} on remote stream being called`)
              const element = document.getElementById(`janusVideo${i}`)
              Janus.attachMediaStream(element, stream)
  
            }.bind(this),
          })
      }
                    // end test
                    },
                  error: function (error) { console.log(error) },

                    })
              }
            }});
    },
  }
}

       
  
</script>

// <style scoped lang="sass">
// .janus-video
//   width: 350px
//   height: 200px
//   background: black
//   border: 5px solid rgba(35, 177, 104, 0.83)
//   margin: 30px 10px
// </style>

<style scoped lang="sass">
.janus-video
  width: 200px
  height: 115px
  background: black
  border: 3px solid #666
  margin: 5px 10px
</style>
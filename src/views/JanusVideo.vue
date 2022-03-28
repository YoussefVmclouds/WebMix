<template>
  <!-- Janus Video -->
  <div>
    <div
    :key="camera"
      v-for="(camera, i) in cameras">
    <video
      :id="`janusVideo${i}`"
      class="janus-video"
      playsinline
      autoplay
      controls
      muted
    ></video>
    <!-- <audio
    :id="`janusAudio${i}`"
    :ref="`janusAudio${i}`"
    :src="`janusVideo${i}`"
    autoplay
    muted="false"
    ></audio> -->
    <!-- <av-bars :ref-link="`janusAudio${i}`" /> -->
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
      cameras: [],
      serverList:[],
      watchID:null,
    }
  },
  mounted () {
    this.initJanus()
  },
  methods: {
    // Init Janus
    initJanus () {
        //  this.janus.attach.bind({x:1})
         this.janus.attach(
          {
            opaqueId: 'test-',

            plugin: 'janus.plugin.streaming',
            success:  (pluginHandle) =>{
            // console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)
              if (pluginHandle) {
                // console.log('pluginHandle=', pluginHandle)


                let body = { 'request': 'list' }
                // let body = { 'request': 'watch', id: parseInt('123') }

                // console.log(`iteration ${i} sending watch request::`)
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
        // console.log(`this is iteration ${i.id} of the for loop`)
        this.janus.attach(
          {
            opaqueId: 'test-' + i,

            plugin: 'janus.plugin.streaming',
            success: function (pluginHandle) {
              // console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)
              if (pluginHandle) {
                // console.log('pluginHandle=', pluginHandle)
                vm.streaming.push({ id: i, plugin: pluginHandle })
                // vm.streaming.push({ id: parseInt('1'), plugin: pluginHandle })
                let body = { 'request': 'watch', id: vm.cameras[i] }
                // let body = { 'request': 'watch', id: parseInt('1') }
                // console.log(`iteration ${i} sending watch request::${body}`)
                pluginHandle.send({ 'message': body })
              }
            },
            error: function (error) { console.log(error) },
            onmessage: function (msg, jsep) {
              // console.log(`iteration ${i} new message received! msg::`, msg)
              // console.log(`iteration ${i} the accompanying jsep was::`, jsep)

              if (jsep !== undefined && jsep !== null) {
                // console.log(`iteration ${i} jsep was not null or undefined THIS IS GOOD`)

                const foundStream = vm.streaming.find(s => s.id === i)

                if (jsep.type === 'offer') {
                //   console.log(`iteration ${i} the jsep type was an offer, lets make an answer`)
                  foundStream.plugin.createAnswer(
                    {
                      jsep,
                      media: { audioSend: false, videoSend: false },
                      success: function (jsep) {
                        // console.log(`iteration ${i} sending a message to request the stream starts`)
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
              // const elementAudio = document.getElementById(`janusAudio${i}`)
              Janus.attachMediaStream(element, stream)
              // Janus.attachMediaStream(elementAudio, stream)
            },
          })
      }
                    // end test
                    },
                  error: function (error) { console.log(error) },

                    })
              }
            }});
      // const vm = this
      // for (let i = 0; i < vm.cameras.length; i++) {
      //   // console.log(`this is iteration ${i.id} of the for loop`)
      //   this.janus.attach(
      //     {
      //       opaqueId: 'test-' + i,

      //       plugin: 'janus.plugin.streaming',
      //       success: function (pluginHandle) {
      //         // console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)
      //         if (pluginHandle) {
      //           // console.log('pluginHandle=', pluginHandle)

      //           // vm.streaming.push({ id: vm.cameras[i], plugin: pluginHandle })
      //           vm.streaming.push({ id: parseInt('1'), plugin: pluginHandle })

      //           // let body = { 'request': 'watch', id: i }
      //           let body = { 'request': 'watch', id: parseInt('1') }

      //           // console.log(`iteration ${i} sending watch request::${body}`)

      //           pluginHandle.send({ 'message': body })
      //         }
      //       },
      //       error: function (error) { console.log(error) },
      //       onmessage: function (msg, jsep) {
      //         // console.log(`iteration ${i} new message received! msg::`, msg)
      //         // console.log(`iteration ${i} the accompanying jsep was::`, jsep)

      //         if (jsep !== undefined && jsep !== null) {
      //           // console.log(`iteration ${i} jsep was not null or undefined THIS IS GOOD`)

      //           const foundStream = vm.streaming.find(s => s.id === i)

      //           if (jsep.type === 'offer') {
      //           //   console.log(`iteration ${i} the jsep type was an offer, lets make an answer`)
      //             foundStream.plugin.createAnswer(
      //               {
      //                 jsep,
      //                 media: { audioSend: false, videoSend: false },
      //                 success: function (jsep) {
      //                   // console.log(`iteration ${i} sending a message to request the stream starts`)
      //                   const body = { 'request': 'start' }

      //                   foundStream.plugin.send({ 'message': body, 'jsep': jsep })
      //                 },
      //                 error: function (error) {
      //                   Janus.error('WebRTC error:', error)
      //                 }
      //               }
      //             )
      //           }
      //         }
      //       },
      //       onremotestream: function (stream) {
      //       //   console.log(`iteration ${i} on remote stream being called`)
      //         const element = document.getElementById(`janusVideo${i}`)
      //         Janus.attachMediaStream(element, stream)
      //       },
      //     })
      // }
    }
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
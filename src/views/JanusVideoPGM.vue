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
  mounted () {
    this.initJanus()
  },
  methods: {
    //test
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
                      this.watchID = this.$route.query.pgm
                        console.log("Looking for (",this.watchID, ") as PGM view")
                pluginHandle.send({ 'message': body,
                  success: (result)=> {
                    this.serverList=result.list
                    this.serverList.forEach(element => {
                    if(this.watchID !=null){
                      if(element.description.search(this.watchID)>-1){
                        this.cameraPGM.push(element.id)
                      }
                    }
                    });
                    // test
  const vm = this
for (let i = 0; i < vm.cameraPGM.length; i++) {
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
                let body = { 'request': 'watch', id: vm.cameraPGM[i] }
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
              const element = document.getElementById(`janusVideoPGM`)
              Janus.attachMediaStream(element, stream)
            },
          })
      }
                    // end test
                    },
                  error: function (error) { console.log(error) },

                    })
              }
            }});
      const vm = this
      for (let i = 0; i < vm.cameraPGM.length; i++) {
        // console.log(`this is iteration ${i.id} of the for loop`)
        this.janus.attach(
          {
            opaqueId: 'test-' + i,

            plugin: 'janus.plugin.streaming',
            success: function (pluginHandle) {
              // console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)
              if (pluginHandle) {
                // console.log('pluginHandle=', pluginHandle)

                // vm.streaming.push({ id: vm.cameras[i], plugin: pluginHandle })
                vm.streaming.push({ id: parseInt('1'), plugin: pluginHandle })

                // let body = { 'request': 'watch', id: i }
                let body = { 'request': 'watch', id: parseInt('1') }

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
              Janus.attachMediaStream(element, stream)
            },
          })
      }
    }

    //end test
    //Orignial
    // Init Janus
  //   initJanus () {
  //        this.janus.attach(
  //         {
  //           opaqueId: 'test-',

  //           plugin: 'janus.plugin.streaming',
  //           success: function (pluginHandle) {
  //             // console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)
  //             if (pluginHandle) {
  //               // console.log('pluginHandle=', pluginHandle)


  //               let body = { 'request': 'list' }
  //               // let body = { 'request': 'watch', id: parseInt('123') }

  //               // console.log(`iteration ${i} sending watch request::`)

  //               pluginHandle.send({ 'message': body,success: function(result) {console.log(result)} })
  //             }
  //           }});
  //     const vm = this
  //     for (let i = 0; i < this.cameraPGM.length; i++) {
  //       // console.log(`this is iteration ${i} of the for loop`)
  //       this.janus.attach(
  //         {
  //           opaqueId: 'test-' + i,

  //           plugin: 'janus.plugin.streaming',
  //           success: function (pluginHandle) {
  //             // console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)
  //             if (pluginHandle) {
  //               // console.log('pluginHandle=', pluginHandle)

  //               vm.streaming.push({ id: i, plugin: pluginHandle })

  //               // let body = { 'request': 'watch', id: i }
  //               let body = { 'request': 'watch', id: parseInt('123') }

  //               // console.log(`iteration ${i} sending watch request::`)

  //               pluginHandle.send({ 'message': body })
  //             }
  //           },
  //           error: function (error) { console.log(error) },
  //           onmessage: function (msg, jsep) {
  //             // console.log(`iteration ${i} new message received! msg::`, msg)
  //             // console.log(`iteration ${i} the accompanying jsep was::`, jsep)

  //             if (jsep !== undefined && jsep !== null) {
  //               // console.log(`iteration ${i} jsep was not null or undefined THIS IS GOOD`)

  //               const foundStream = vm.streaming.find(s => s.id === i)

  //               if (jsep.type === 'offer') {
  //               //   console.log(`iteration ${i} the jsep type was an offer, lets make an answer`)
  //                 foundStream.plugin.createAnswer(
  //                   {
  //                     jsep,
  //                     media: { audioSend: false, videoSend: false },
  //                     success: function (jsep) {
  //                       // console.log(`iteration ${i} sending a message to request the stream starts`)
  //                       const body = { 'request': 'start' }

  //                       foundStream.plugin.send({ 'message': body, 'jsep': jsep })
  //                     },
  //                     error: function (error) {
  //                       Janus.error('WebRTC error:', error)
  //                     }
  //                   }
  //                 )
  //               }
  //             }
  //           },
  //           onremotestream: function (stream) {
  //           //   console.log(`iteration ${i} on remote stream being called`)
  //             const element = document.getElementById(`janusVideoPGM`)
  //             Janus.attachMediaStream(element, stream)
  //           },
  //         })
  //     }
  //   }
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
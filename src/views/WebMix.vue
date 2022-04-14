<template>
<v-row >
  <!-- Live PGM window + Layout controls + video feeds -->
  <!-- PGM window -->
<v-col
align="center" justify="center"
cols='7'
>
<!-- Live PGM window -->
<v-card 
color= "#000"
width="92%"
height="53vh"
>
<v-card-title class="justify-center">
<center id="livePGMContianer">
      <div v-if="!loadingPGM">
        <JanusVideoPGM :janus="janusPGM" />
    </div>
</center>
</v-card-title>
</v-card>

<!-- Live Layouts -->
<v-card 
class="mt-5 mb-5"
color="secondary"
width="70%"
height="5vh"
>
<v-radio-group
  class="justify-center"
  v-model="PGMlayouts"
  row
>
  <v-spacer></v-spacer>

  <v-radio
    color="#ff0000"
    value="1x1"
  ><template v-slot:label>
         <v-img width="65" height="4.1vh" src="../assets/images/layouts/1x1.png"/>
    </template>
    </v-radio>
    <v-spacer></v-spacer>

  <v-radio
    color="#ff0000"
    value="1x2"
  ><template v-slot:label>
         <v-img  width="65" height="4.1vh" src="../assets/images/layouts/1x2.png"/>
    </template>
    </v-radio>
      <v-spacer></v-spacer>
  <v-radio
    color="#ff0000"
    value="1x3"
  ><template v-slot:label>
         <v-img width="65" height="4.1vh" src="../assets/images/layouts/1x3.png"/>
    </template>
    </v-radio>
      <v-spacer></v-spacer>
  <v-radio
    color="#ff0000"
    value="2x2"
  ><template v-slot:label>
         <v-img width="65" height="4.1vh" src="../assets/images/layouts/2x2.png"/>
    </template>
    </v-radio>
      <v-spacer></v-spacer>
</v-radio-group>
</v-card>

<!-- Live cameras feeds -->
<v-card 
color="#000"
height="32vh"
>
<v-card-title class="justify-center">
<div id="prvAreaJanus">
    <div v-if="!loading">
            <JanusVideo :janus="janus" />
    </div>
</div>
</v-card-title>
</v-card>
</v-col>

<!-- CG live keys -->
<v-col
cols='1'
>
<v-card 
color="secondary"
height="95vh"
>
<v-card-title class="justify-center">
</v-card-title>
<v-card-subtitle>Status</v-card-subtitle>
<v-col
cols="12">
<h4 class="ml-3">Live 
    <v-icon
    class=""
    normal
    color="#00FF00"
    v-if="this.liveIcon">
      mdi-moon-full
    </v-icon>
    <v-icon
    class=""
    normal
    v-else>
      mdi-moon-new
    </v-icon>
    </h4>
  <h4 class="ml-3 mb-3">Rec.   
    <v-icon
    class=""
    normal
    color="#00FF00"
    v-if="this.recordIcon">
      mdi-moon-full
    </v-icon>
    <v-icon
    class=""
    normal
    v-else>
      mdi-moon-new
    </v-icon>
    </h4>
</v-col>
    <v-divider></v-divider>
    <v-card-subtitle>CG Keys</v-card-subtitle>
 <v-switch
    v-model="bgKey"
    :disabled="bgUploadIcon == 0"
    class="pl-2"
    label="BG"
    color="#FF0000"
    hide-details
  ></v-switch>
<v-switch
    v-model="logoKey"
    :disabled="logoUploadIcon == 0"
    class="pl-2"
    label="Logo"
    color="#FF0000"
    hide-details
  ></v-switch>
 <v-switch
    v-model="bugKey"
    :disabled="bugUploadIcon == 0"
    class="pl-2 mb-5"
    label="Bug"
    color="#FF0000"
    hide-details
  ></v-switch>
<v-divider></v-divider>
 <v-switch
    v-model="s01Key"
    :disabled="straps[0].liveReady==0"
    class="pl-2"
    label="S-01"
    color="#FF0000"
    hide-details
  ></v-switch>
 <v-switch
    v-model="s02Key"
    :disabled="straps[1].liveReady==0"
    class="pl-2"
    label="S-02"
    color="#FF0000"
    hide-details
  ></v-switch>
 <v-switch
    v-model="s03Key"
    :disabled="straps[2].liveReady==0"
    class="pl-2 mb-5"
    label="S-03"
    color="#FF0000"
    hide-details
  ></v-switch>
<v-divider></v-divider>

</v-card>
</v-col>
<!-- End of CG live keys -->

<!-- Managment section window -->
<v-col
cols='4'
>
<v-card 
color="#000"
height="95vh"
class="rightMenuContainer"
>
<v-card-title class="justify-center">
	GFX Manager
</v-card-title>
    <v-list class="theme--dark">
      <!-- BG/logo/bug -->
      <v-list-group
		class="rightMenu"
        active-class="rightMenu-active"
		color= "white"
        :value="false"
		prepend-icon="mdi-watermark"
        no-action
        sub-group
      >
        <template v-slot:activator>
          <v-list-item-content>
            <v-list-item-title>
              <p class="font-weight-bold">
                Background / Logo / Bug
                </p>
                </v-list-item-title>
          </v-list-item-content>
        </template>

        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Background</v-list-item-title>
				<v-list-item-subtitle>
          <v-row>
            <v-col
            cols="8">
					<v-file-input
						accept=".png"
						label="single .png image"
						filled
						class="mt-3"
            @change="bgUpload"
            @click:clear="bgUploadIcon=false; bgKey=false;"
						prepend-icon="mdi-camera"
					></v-file-input>
            </v-col>
            <v-col
            cols="4">
          <v-icon
          class="mt-5"
          x-large
          v-if="this.bgUploadIcon == true">
            mdi-check-bold
          </v-icon>
          <v-icon
          class="mt-5"
          x-large
          v-else>
            mdi-close-thick
          </v-icon>
            </v-col>
          </v-row>
				</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
		        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Logo</v-list-item-title>
            <v-list-item-subtitle>
                        <v-row>
            <v-col
            cols="8">
					<v-file-input
						accept=".png"
						label="single .png image"
						filled
						class="mt-3"
            @change="logoUpload"
            @click:clear="logoUploadIcon=false; logoKey=false;"
						prepend-icon="mdi-camera"
					></v-file-input>
            </v-col>
            <v-col
            cols="4">
          <v-icon
          class="mt-5"
          x-large
          v-if="this.logoUploadIcon == true">
            mdi-check-bold
          </v-icon>
          <v-icon
          class="mt-5"
          x-large
          v-else>
            mdi-close-thick
          </v-icon>
            </v-col>
          </v-row>
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
		        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Bug</v-list-item-title>
                        <v-list-item-subtitle>
                        <v-row>
            <v-col
            cols="8">
					<v-file-input
						accept=".png"
						label="single .png image"
						filled
						class="mt-3"
            @change="bugUpload"
            @click:clear="bugUploadIcon=false; bugKey=false;"
						prepend-icon="mdi-camera"
					></v-file-input>
            </v-col>
            <v-col
            cols="4">
          <v-icon
          class="mt-5"
          x-large
          v-if="this.bugUploadIcon == true">
            mdi-check-bold
          </v-icon>
          <v-icon
          class="mt-5"
          x-large
          v-else>
            mdi-close-thick
          </v-icon>
            </v-col>
          </v-row>
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </v-list-group>
      <!-- End BG/logo/bug -->
      <!-- Straps -->
	<v-list-group
		class="rightMenu"
        active-class="rightMenu-active"
        color= "white"
        :value="false"
        prepend-icon="mdi-text-box-multiple"
        no-action
        sub-group
      >
        <template v-slot:activator>
          <v-list-item-content>
            <v-list-item-title>
              <p class="font-weight-bold">
                Straps
              </p>
            </v-list-item-title>
          </v-list-item-content>
        </template>

        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
                  <v-tabs
                  v-model="tab"
                  background-color="transparent"
                  color="#000"
                  grow
                >
                  <v-tab
                    v-for="strap in straps"
                    :key="strap.id"
                  >
                    {{ strap.title }}
                  </v-tab>
                </v-tabs>

                <v-tabs-items v-model="tab">
                  <v-tab-item
                    v-for="strap in straps"
                    :key="strap.id"
                  >
                    <v-card
                      flat
                    >
                    <v-textarea
                      dark
                      class="strapTextArea mt-5 text--black mx-1"
                      color="#000"
                      v-model="strap.textArea"
                      outlined
                      shaped
                      filled
                      rows="1"
                      row-height="15" 
                      label="Strap Text"
                    ></v-textarea>
                    <v-row justify="space-around"><h3 style="color:black !important;">Text Color</h3></v-row>
                    <v-row justify="space-around">
                    <v-color-picker
                        v-model="strap.textColor"
                        style="background:grey;color:white"
                        hide-canvas
                        hide-mode-switch
                        flat
                      >
                      </v-color-picker>
                    </v-row>
                    <v-row justify="space-around">
                     <v-select
                      class="fontSelect mx-5 mt-5"
                      :items="strapFonts"
                      v-model="strap.textFont"
                      label="Font Famliy"
                      outlined
                    ></v-select>
                    </v-row>
                    <v-row justify="space-around"><h3 style="color:black !important;">Background Color</h3></v-row>
                    <v-row justify="space-around">
                    <v-color-picker
                        v-model="strap.bgColor"
                        style="background:grey;color:white"
                        hide-canvas
                        hide-mode-switch
                        flat
                      >
                      </v-color-picker>
                    </v-row>
                    <v-row justify="space-around">
                    <v-btn
                      color="success"
                      class="my-5 white--text"
                      @click="strapUpdate"
                    >
                      Update
                      <v-icon
                        right
                        light
                      >
                        mdi-cached
                      </v-icon>
                    </v-btn>
                    </v-row>
                    </v-card>
                  </v-tab-item>
                </v-tabs-items>
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list-group>
        <!-- End Straps -->
<!-- Camera Order -->
	<v-list-group
		class="rightMenu"
        active-class="rightMenu-active"
		color= "white"
		:value="true"
		prepend-icon="mdi-tune-vertical"
        no-action
        sub-group
      >
        <template v-slot:activator>
          <v-list-item-content>
            <v-list-item-title>
              <p class="font-weight-bold">
              Cameras Order
              </p>  
            </v-list-item-title>
          </v-list-item-content>
        </template>

        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Layout 1x1</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
		        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Layout 1x2</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
		        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Layout 1x3</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>Layout 2x2</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list-group>
<!-- End Audio Mixer -->
	  
    </v-list>

</v-card>
</v-col>
<!-- End of Managment section -->
</v-row>
</template>

<script>

// eslint-disable-next-line object-curly-newline
import { mdiPoll, mdiLabelVariantOutline, mdiCurrencyUsd, mdiHelpCircleOutline } from '@mdi/js'
import Janus from './janus'


export default {
  components: {
    JanusVideo: () => import('./JanusVideo'),
    JanusVideoPGM: () => import('./JanusVideoPGM'),
  },
  
  mounted() {  
    document.title = "WebMixer";  
    // Janus Test
    this.initJanus();
    this.initJanusPGM();
    // End Janus

	  // test Drag and Drop
	  window.addEventListener('load', (event) => {
    var dragSrcEl = null;
    function handleDragStart(e) {
      this.style.opacity = '0.4';
      dragSrcEl = this;
      e.dataTransfer.effectAllowed = 'move';
      e.dataTransfer.setData('text/html', this.innerHTML);
    }
    function handleDragOver(e) {
      if (e.preventDefault) {
        e.preventDefault();
      }
  
      e.dataTransfer.dropEffect = 'move';
      
      return false;
    }
  
    function handleDragEnter(e) {
      this.classList.add('over');
    }
  
    function handleDragLeave(e) {
      this.classList.remove('over');
    }
  
    function handleDrop(e) {
      if (e.stopPropagation) {
        e.stopPropagation(); // stops the browser from redirecting.
      }
      
      if (dragSrcEl != this) {
        dragSrcEl.innerHTML = this.innerHTML;
        this.innerHTML = e.dataTransfer.getData('text/html');
      }
      
      return false;
    }
  
    function handleDragEnd(e) {
      this.style.opacity = '1';
      
      items.forEach(function (item) {
        item.classList.remove('over');
      });
    }
    
    
    let items = document.querySelectorAll('.prvFeedDrag');
    items.forEach(function(item) {
      item.addEventListener('dragstart', handleDragStart, false);
      item.addEventListener('dragenter', handleDragEnter, false);
      item.addEventListener('dragover', handleDragOver, false);
      item.addEventListener('dragleave', handleDragLeave, false);
      item.addEventListener('drop', handleDrop, false);
      item.addEventListener('dragend', handleDragEnd, false);
    });
  });
	// end test of drag and drop
  },  
  beforeDestroy () {
    this.janus.destroy()
    this.janusPGM.destroy()
  },
  setup() {
    return {
    }
  },
  data() {
		return {
      // right menu fields
      bgUploadIcon: false,
      logoUploadIcon: false,
      bugUploadIcon: false,
      tab: null,
      straps: [
        {id: 1, title:'Strap-01', textArea: 'Test S1', liveReady: true, textColor: '', textFont: '', bgColor:''},
        {id: 2, title:'Strap-02', textArea: '', liveReady: false, textColor: '', textFont: '', bgColor:''},
        {id: 3, title:'Strap-03', textArea: '', liveReady: false, textColor: '', textFont: '', bgColor:''},
      ],
      tempFonts:['Arial', 'Sans-bold', 'Sans-normal', 'Aja-italic','test-test'],
      strapFonts: [],
      // CG Keys section
      liveIcon: false,
      recordIcon: false,
      bgKey: false,
      logoKey: false,
      bugKey: false,
      s01Key: false,
      s02Key: false,
      s03Key: false,
      // Janus Test
      janus: null,
      janusPGM: null,
      loading: false,
      loadingPGM: false,
      // End Janus
      // Live Layout
      PGMlayouts: '1x1',
		};
	},
  watch: {
    liveIcon(newValue){
      if(newValue==true){
        console.log("Now you are live: ",newValue);
      }
      else{
        console.log("You are no longer Live! ",newValue);
      }
    },
    recordIcon(newValue){
      if(newValue==true){
        console.log("Live recording is on: ",newValue);
      }
      else{
        console.log("Live recording is off: ",newValue);
      }
    },
    bgKey(newValue){
      if(newValue==true){
        console.log("Turning bgKey on: ",newValue);
      }
      else{
        console.log("Turning bgKey off: ",newValue);
      }
    },
    logoKey(newValue){
      if(newValue==true){
        console.log("Turning logoKey on: ",newValue);
      }
      else{
        console.log("Turning logoKey off: ",newValue);
      }
    },
    bugKey(newValue){
      if(newValue==true){
        console.log("Turning bugKey on: ",newValue);
      }
      else{
        console.log("Turning bugKey off: ",newValue);
      }
    },
    s01Key(newValue){
      if(newValue==true){
        console.log("Turning s01Key on: ",newValue);
      }
      else{
        console.log("Turning s01Key off: ",newValue);
      }
      },
    s02Key(newValue){
      if(newValue==true){
        console.log("Turning s02Key on: ",newValue);
      }
      else{
        console.log("Turning s02Key off: ",newValue);
      }
    },
    s03Key(newValue){
      if(newValue==true){
        console.log("Turning s03Key on: ",newValue);
      }
      else{
        console.log("Turning s03Key off: ",newValue);
      }
    },
    PGMlayouts(newValue){
        console.log("PGM Layout Changed to: ",newValue);
    },
  },
	computed: {
	},
	created() {
    this.loadStrapFonts();
    // dummy live and rec icons
	setTimeout(() => {
			this.liveIcon=true;
      this.recordIcon=true;
		}, 5000);	
  
    },
	methods: {
    bgUpload(e){
      // upload background image
      var files = e;
      if (!files.length) {
        console.log("Trying to upload background image name: ",files.name);
        this.bgUploadIcon=true;
        return;
      }  
      },
    logoUpload(e){
      // upload logo image
      var files = e;
      if (!files.length) {
        console.log("Trying to upload logo image: ",files.name);
        this.logoUploadIcon=true;
        return;
      }  
    },
    bugUpload(e){
      // upload bug image
      var files = e;
      if (!files.length) {
        console.log("Trying to upload bug image: ",files.name);
        this.bugUploadIcon=true;
        return;
      }  
    },
    loadStrapFonts(){
      this.tempFonts.forEach(element => {
				this.strapFonts.push(element);
			});
    },
    strapUpdate(){
      this.straps.forEach(strap => {
        if(strap.textArea.length>0){
          strap.liveReady=true;
        }
        if(strap.textArea.length==0){
          strap.liveReady=false;
          if(strap.id==1){
            this.s01Key=false;
          }
          if(strap.id==2){
            this.s02Key=false;
          }
          if(strap.id==3){
            this.s03Key=false;
          }
        }
      });
      console.log("Trying to update a Strapes: ",this.straps);
    },
    initJanus () {
        this.loading = true
        // test server
        let server = 'https://janus.conf.meetecho.com/janus'
        // vmcloudserver
        // let server = 'wss://hlsdvr.vmclouds.co.uk:8989'
        // let server = 'http://hlsdvr.vmclouds.co.uk/janus'
        Janus.init({
          callback: () => {
            this.janus = new Janus(
              {
                server,
                success: () => {
                  this.loading = false
                },
                error: (cause) => {
                  console.log(cause)
                },
                destroyed: () => {
                  console.log('janus init destroyed')
                }
              })
          }
        })
      },
    initJanusPGM () {
        this.loadingPGM = true
        // test server
        let server = 'https://janus.conf.meetecho.com/janus'
        // vmcloudserver
        // let server = 'wss://hlsdvr.vmclouds.co.uk:8989'
          Janus.init({
          callback: () => {
            this.janusPGM = new Janus(
              {
                server,
                success: () => {
                  this.loadingPGM = false
                },
                error: (cause) => {
                  console.log(cause)
                },
                destroyed: () => {
                  console.log('janus init destroyed')
                }
              })
          }
        })
      },
	  },
  }
</script>

<style>

#livePGMContianer{
	margin: auto;
	/* width: 80vw; */
	height: 60vh;
  overflow: hidden;
}
/* works with Janus component */
#livePGMContianer div div div video{
/* Make video to at least 100% wide and tall */
  min-width: 96%; 
  min-height: auto; 

  /* Setting width & height to auto prevents the browser from stretching or squishing the video */
  width: auto;
  height: auto;

  /* Center the video */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
}
#prvAreaJanus{
    display: flex;
    overflow-x: auto;
	padding-bottom: 5vh;
}
#prvAreaJanus div div{
	display: flex !important;
	/* margin-left: 2vw; */
  	/* cursor: move; */
}
/* #prvAreaJanus div{
  	display: flex !important;
} */
.prvFeedDrag{
	border: 3px solid #666;
  	border-radius: .5em;
}
.rightMenu{
background-color: black;
color:white;
}
/* .rightMenu-active{
background-color: white;
color:black;
} */
.rightMenuContainer{
  overflow-y: auto;
}
.v-list-group--active{
	background-color: grey;
color:black !important;
}
.mdi-close-thick{
  color: red  !important;
}
.mdi-check-bold{
  color: green !important;
}
.strapTextArea textarea {
  color: #000 !important;
}
.fontSelect *{
  color: #000 !important;
}
.fontSelect div{
  color: #000 !important;
}
.v-menu__content div{
  color:#000 !important;
}

</style>



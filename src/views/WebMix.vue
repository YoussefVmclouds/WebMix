<template>
  <v-row>
    <!-- Live PGM window + Layout controls + video feeds -->
    <!-- PGM window -->
    <v-col align="center" justify="center" cols="7">
      <!-- Live PGM window -->
      <v-card color="#000" width="92%" height="53vh">
        <v-card-title class="justify-center">
          <center id="livePGMContianer">
            <div v-if="!loadingPGM">
              <JanusVideoPGM :janus="janusPGM" />
            </div>
          </center>
        </v-card-title>
      </v-card>

      <!-- Live Layouts -->
      <v-card class="mt-5 mb-5" color="secondary" width="70%" height="5vh">
        <v-radio-group class="justify-center" v-model="PGMlayouts" row>
          <v-spacer></v-spacer>

          <v-radio color="#ff0000" value="1x1"
            ><template v-slot:label>
              <v-img
                contain
                width="4vw"
                height="auto"
                src="../assets/images/layouts/1x1.png"
              />
            </template>
          </v-radio>
          <v-spacer></v-spacer>

          <v-radio color="#ff0000" value="1x2"
            ><template v-slot:label>
              <v-img
                contain
                width="4vw"
                height="auto"
                src="../assets/images/layouts/1x2.png"
              />
            </template>
          </v-radio>
          <v-spacer></v-spacer>
          <v-radio color="#ff0000" value="1x3"
            ><template v-slot:label>
              <v-img
                contain
                width="4vw"
                height="auto"
                src="../assets/images/layouts/1x3.png"
              />
            </template>
          </v-radio>
          <v-spacer></v-spacer>
          <v-radio color="#ff0000" value="2x2"
            ><template v-slot:label>
              <v-img
                contain
                width="4vw"
                height="auto"
                src="../assets/images/layouts/2x2.png"
              />
            </template>
          </v-radio>
          <v-spacer></v-spacer>
        </v-radio-group>
      </v-card>

      <!-- Live cameras feeds -->
      <v-card color="#000" height="32vh">
        <v-card-title class="justify-center">
          <div id="prvAreaJanus">
            <div v-if="!loading">
              <JanusVideo :janus="janus" ref="jvt" @main-camera="mainCamera" />
              <!-- <JVT :janus="janus" ref="jvt" /> -->
            </div>
          </div>
        </v-card-title>
      </v-card>
    </v-col>

    <!-- CG live keys -->
    <v-col cols="1">
      <v-card color="secondary" height="95vh">
        <h3 class="mx-1 my-2 font-weight-bold">Status</h3>
        <v-col cols="12">
          <div class="mx-auto">
            Live
            <v-icon class="" normal color="#00FF00" v-if="this.liveIcon">
              mdi-moon-full
            </v-icon>
            <v-icon class="" normal v-else> mdi-moon-new </v-icon>
          </div>
          <div class="mx-auto">
            Rec.
            <v-icon class="" normal color="#00FF00" v-if="this.recordIcon">
              mdi-moon-full
            </v-icon>
            <v-icon class="" normal v-else> mdi-moon-new </v-icon>
          </div>
        </v-col>
        <v-divider></v-divider>
        <h3 class="mx-1 my-2 font-weight-bold">CG Keys</h3>
        <div>
          <v-switch
            v-model="bgKey"
            :disabled="bgUploadIcon == 0"
            style="display: inline-block"
            size="100"
            class="pl-1 mx-auto"
            color="#FF0000"
            hide-details
          ></v-switch>
          <div style="display: inline-block">BG</div>
        </div>
        <div>
          <v-switch
            v-model="logoKey"
            :disabled="logoUploadIcon == 0"
            style="display: inline-block"
            class="pl-1 mx-auto"
            label=""
            color="#FF0000"
            hide-details
          ></v-switch>
          <div style="display: inline-block">Logo</div>
        </div>
        <div>
          <v-switch
            v-model="bugKey"
            :disabled="bugUploadIcon == 0"
            style="display: inline-block"
            class="pl-1 mx-auto mb-5"
            label=""
            color="#FF0000"
            hide-details
          ></v-switch>
          <div style="display: inline-block">Bug</div>
        </div>

        <v-divider></v-divider>
        <div>
          <v-switch
            v-model="s01Key"
            :disabled="straps[0].liveReady == 0"
            style="display: inline-block"
            class="pl-1 mx-auto"
            label=""
            color="#FF0000"
            hide-details
          ></v-switch>
          <div style="display: inline-block">S-01</div>
        </div>
        <div>
          <v-switch
            v-model="s02Key"
            :disabled="straps[1].liveReady == 0"
            style="display: inline-block"
            class="pl-1 mx-auto"
            label=""
            color="#FF0000"
            hide-details
          ></v-switch>
          <div style="display: inline-block">S-02</div>
        </div>
        <div>
          <v-switch
            v-model="s03Key"
            :disabled="straps[2].liveReady == 0"
            style="display: inline-block"
            class="pl-1 mx-auto mb-5"
            label=""
            color="#FF0000"
            hide-details
          ></v-switch>
          <div style="display: inline-block">S-03</div>
        </div>

        <v-divider></v-divider>
        <!-- Invite Link Button -->
        <v-btn
          @click="showInviteUrl = !showInviteUrl"
          class="my-3 mx-auto"
          rounded
          color="success"
        >
          <v-icon left> mdi-share </v-icon>
          Invite
        </v-btn>

        <v-divider></v-divider>
      </v-card>
    </v-col>
    <!-- End of CG live keys -->

    <!-- Managment section window -->
    <v-col cols="4">
      <v-card color="#000" height="95vh" class="rightMenuContainer">
        <v-card-title class="justify-center"> GFX Manager </v-card-title>
        <v-list class="theme--dark">
          <!-- BG/logo/bug -->
          <v-list-group
            class="rightMenu"
            active-class="rightMenu-active"
            color="white"
            :value="false"
            prepend-icon="mdi-watermark"
            no-action
            sub-group
          >
            <template v-slot:activator>
              <v-list-item-content>
                <v-list-item-title>
                  <p class="font-weight-bold">Background / Logo / Bug</p>
                </v-list-item-title>
              </v-list-item-content>
            </template>

            <v-list-item style="padding-left: 3vw">
              <v-list-item-content>
                <v-list-item-title>Background</v-list-item-title>
                <v-list-item-subtitle>
                  <v-row>
                    <v-col cols="8">
                      <v-file-input
                        accept=".png"
                        label="single .png img"
                        filled
                        class="mt-3"
                        @change="bgUpload"
                        @click:clear="
                          bgUploadIcon = false;
                          bgKey = false;
                        "
                        prepend-icon="mdi-camera"
                      ></v-file-input>
                    </v-col>
                    <v-col cols="4">
                      <v-icon class="mt-5" v-if="this.bgUploadIcon == true">
                        mdi-check-bold
                      </v-icon>
                      <v-icon class="mt-5" v-else> mdi-close-thick </v-icon>
                    </v-col>
                  </v-row>
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item style="padding-left: 3vw">
              <v-list-item-content>
                <v-list-item-title>Logo</v-list-item-title>
                <v-list-item-subtitle>
                  <v-row>
                    <v-col cols="8">
                      <v-file-input
                        accept=".png"
                        label="single .png img"
                        filled
                        class="mt-3"
                        @change="logoUpload"
                        @click:clear="
                          logoUploadIcon = false;
                          logoKey = false;
                        "
                        prepend-icon="mdi-camera"
                      ></v-file-input>
                    </v-col>
                    <v-col cols="4">
                      <v-icon class="mt-5" v-if="this.logoUploadIcon == true">
                        mdi-check-bold
                      </v-icon>
                      <v-icon class="mt-5" v-else> mdi-close-thick </v-icon>
                    </v-col>
                  </v-row>
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item style="padding-left: 3vw">
              <v-list-item-content>
                <v-list-item-title>Bug</v-list-item-title>
                <v-list-item-subtitle>
                  <v-row>
                    <v-col cols="8">
                      <v-file-input
                        accept=".png"
                        label="single .png img"
                        filled
                        class="mt-3"
                        @change="bugUpload"
                        @click:clear="
                          bugUploadIcon = false;
                          bugKey = false;
                        "
                        prepend-icon="mdi-camera"
                      ></v-file-input>
                    </v-col>
                    <v-col cols="4">
                      <v-icon class="mt-5" v-if="this.bugUploadIcon == true">
                        mdi-check-bold
                      </v-icon>
                      <v-icon class="mt-5" v-else> mdi-close-thick </v-icon>
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
            color="white"
            :value="false"
            prepend-icon="mdi-text-box-multiple"
            no-action
            sub-group
          >
            <template v-slot:activator>
              <v-list-item-content>
                <v-list-item-title>
                  <p class="font-weight-bold">Straps</p>
                </v-list-item-title>
              </v-list-item-content>
            </template>

            <v-list-item style="padding-left: 1vw">
              <v-list-item-content>
                <v-list-item-title>
                  <v-tabs
                    v-model="strapTab"
                    background-color="transparent"
                    color="#000"
                    grow
                  >
                    <v-tab v-for="strap in straps" :key="strap.id" class="py-5">
                      {{ strap.title }}
                    </v-tab>
                  </v-tabs>

                  <v-tabs-items v-model="strapTab">
                    <v-tab-item v-for="strap in straps" :key="strap.id">
                      <v-card flat>
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
                        <v-row justify="space-around"
                          ><h3 style="color: black !important">
                            Text Color
                          </h3></v-row
                        >
                        <v-row justify="space-around">
                          <v-color-picker
                            v-model="strap.textColor"
                            style="background: grey; color: white"
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
                        <v-row justify="space-around"
                          ><h3 style="color: black !important">
                            Background Color
                          </h3></v-row
                        >
                        <v-row justify="space-around">
                          <v-color-picker
                            v-model="strap.bgColor"
                            style="background: grey; color: white"
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
                            <v-icon right light> mdi-cached </v-icon>
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
            color="white"
            :value="true"
            prepend-icon="mdi-video"
            no-action
            sub-group
          >
            <template v-slot:activator>
              <v-list-item-content>
                <v-list-item-title>
                  <p class="font-weight-bold">Cameras Order</p>
                </v-list-item-title>
              </v-list-item-content>
            </template>

            <v-list-item style="padding-left: 1vw">
              <v-list-item-content>
                <v-list-item-title>
                  <v-tabs
                    v-model="layoutTab"
                    background-color="transparent"
                    color="#000"
                    grow
                  >
                    <v-tab v-for="layout in layouts" :key="layout.id">
                      <strong>{{ layout.title }}</strong>
                    </v-tab>
                  </v-tabs>

                  <v-tabs-items v-model="layoutTab">
                    <v-tab-item v-for="layout in layouts" :key="layout.id">
                      <v-card flat>
                        <v-row class="mt-5" justify="space-around"> </v-row>
                        <v-row justify="space-around">
                          <template v-if="$refs.jvt != null">
                            <v-col cols="12">
                              <v-select
                                v-for="cam in layout.cams"
                                :key="cam.id"
                                :items="$refs.jvt.cameras"
                                v-model="cam.camSelect"
                                :label="`Select ` + cam.camLabel"
                                class="fontSelect mx-3"
                                dense
                                outlined
                              ></v-select>
                            </v-col>
                          </template>
                        </v-row>
                        <v-row justify="space-around">
                          <v-btn
                            color="warning"
                            class="mb-5 white--text"
                            @click="layoutSet"
                          >
                            Set
                            <v-icon right light> mdi-cached </v-icon>
                          </v-btn>
                        </v-row>
                      </v-card>
                    </v-tab-item>
                  </v-tabs-items>
                </v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-group>
          <!-- End Audio Mixer -->
        </v-list>
      </v-card>
    </v-col>
    <!-- End of Managment section -->

    <!-- Overlay to show Url link on screen for user to take a copy -->
    <v-overlay
      style="
        width: 100vw;
        height: 100vh;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      "
      opacity="0.75"
      :value="showInviteUrl"
    >
      <v-card width="45vw" color="secondary">
        <v-card-title
          ><span class="white--text">Invite Participants</span></v-card-title
        >
        <v-card-text>
          <v-text-field
            light
            color="black"
            v-model="inviteUrl"
            readonly
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="showInviteUrl = !showInviteUrl" tile color="white">
            <span class="black--text">Close</span>
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-overlay>
    <!-- End of Invite Button and Link Overlay -->
  </v-row>
</template>

<script>
// eslint-disable-next-line object-curly-newline
import {
  mdiPoll,
  mdiLabelVariantOutline,
  mdiCurrencyUsd,
  mdiHelpCircleOutline,
} from "@mdi/js";
// import Janus from "./janus";
import { Janus } from "janus-gateway";
import VueResponsiveText from "vue-responsive-text";
import ResizeText from "vue-resize-text";

export default {
  props: ["eventKey"],
  components: {
    JanusVideo: () => import("./JanusVideo"),
    // JVT: () => import('./JVT'),
    JanusVideoPGM: () => import("./JanusVideoPGM"),
    VueResponsiveText,
  },
  directives: {
    ResizeText,
  },
  mounted() {
    document.title = "WebMix";
    // Janus Test
    this.initJanus();
    this.initJanusPGM();
    // End Janus

    // test Drag and Drop
    window.addEventListener("load", (event) => {
      var dragSrcEl = null;
      function handleDragStart(e) {
        this.style.opacity = "0.4";
        dragSrcEl = this;
        e.dataTransfer.effectAllowed = "move";
        e.dataTransfer.setData("text/html", this.innerHTML);
      }
      function handleDragOver(e) {
        if (e.preventDefault) {
          e.preventDefault();
        }

        e.dataTransfer.dropEffect = "move";

        return false;
      }

      function handleDragEnter(e) {
        this.classList.add("over");
      }

      function handleDragLeave(e) {
        this.classList.remove("over");
      }

      function handleDrop(e) {
        if (e.stopPropagation) {
          e.stopPropagation(); // stops the browser from redirecting.
        }

        if (dragSrcEl != this) {
          dragSrcEl.innerHTML = this.innerHTML;
          this.innerHTML = e.dataTransfer.getData("text/html");
        }

        return false;
      }

      function handleDragEnd(e) {
        this.style.opacity = "1";

        items.forEach(function (item) {
          item.classList.remove("over");
        });
      }

      let items = document.querySelectorAll(".prvFeedDrag");
      items.forEach(function (item) {
        item.addEventListener("dragstart", handleDragStart, false);
        item.addEventListener("dragenter", handleDragEnter, false);
        item.addEventListener("dragover", handleDragOver, false);
        item.addEventListener("dragleave", handleDragLeave, false);
        item.addEventListener("drop", handleDrop, false);
        item.addEventListener("dragend", handleDragEnd, false);
      });
    });
    // end test of drag and drop
  },
  beforeDestroy() {
    this.janus.destroy();
    this.janusPGM.destroy();
  },
  setup() {
    return {};
  },
  data() {
    return {
      // right menu fields
      bgUploadIcon: false,
      logoUploadIcon: false,
      bugUploadIcon: false,
      strapTab: null,
      layoutTab: null,
      straps: [
        {
          id: 1,
          title: "Strap-01",
          textArea: "Test S1",
          liveReady: true,
          textColor: "",
          textFont: "",
          bgColor: "",
        },
        {
          id: 2,
          title: "Strap-02",
          textArea: "",
          liveReady: false,
          textColor: "",
          textFont: "",
          bgColor: "",
        },
        {
          id: 3,
          title: "Strap-03",
          textArea: "",
          liveReady: false,
          textColor: "",
          textFont: "",
          bgColor: "",
        },
      ],
      layouts: [
        {
          id: 0,
          title: "1X1",
          cams: [{ id: 1, camLabel: "CAM 1", camSelect: "" }],
        },
        {
          id: 1,
          title: "1X2",
          cams: [
            { id: 1, camLabel: "CAM 1", camSelect: "" },
            { id: 2, camLabel: "CAM 2", camSelect: "" },
          ],
        },
        {
          id: 2,
          title: "1X3",
          cams: [
            { id: 1, camLabel: "CAM 1", camSelect: "" },
            { id: 2, camLabel: "CAM 2", camSelect: "" },
            { id: 3, camLabel: "CAM 3", camSelect: "" },
          ],
        },
        {
          id: 3,
          title: "2X2",
          cams: [
            { id: 1, camLabel: "CAM 1", camSelect: "" },
            { id: 2, camLabel: "CAM 2", camSelect: "" },
            { id: 3, camLabel: "CAM 3", camSelect: "" },
            { id: 4, camLabel: "CAM 4", camSelect: "" },
          ],
        },
      ],
      tempFonts: [
        "Arial",
        "Sans-bold",
        "Sans-normal",
        "Aja-italic",
        "test-test",
      ],
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
      // Active Live Layout
      PGMlayouts: "1x1",
      // Invite Url and show Overlay
      inviteUrl: "https://hlsdvr.vmclouds.co.uk/stream/",
      showInviteUrl: false,
    };
  },
  watch: {
    liveIcon(newValue) {
      if (newValue == true) {
        console.log("Now you are live: ", newValue);
      } else {
        console.log("You are no longer Live! ", newValue);
      }
    },
    recordIcon(newValue) {
      if (newValue == true) {
        console.log("Live recording is on: ", newValue);
      } else {
        console.log("Live recording is off: ", newValue);
      }
    },
    bgKey(newValue) {
      if (newValue == true) {
        console.log("Turning bgKey on: ", newValue);
      } else {
        console.log("Turning bgKey off: ", newValue);
      }
    },
    logoKey(newValue) {
      if (newValue == true) {
        console.log("Turning logoKey on: ", newValue);
      } else {
        console.log("Turning logoKey off: ", newValue);
      }
    },
    bugKey(newValue) {
      if (newValue == true) {
        console.log("Turning bugKey on: ", newValue);
      } else {
        console.log("Turning bugKey off: ", newValue);
      }
    },
    s01Key(newValue) {
      if (newValue == true) {
        console.log("Turning s01Key on: ", newValue);
      } else {
        console.log("Turning s01Key off: ", newValue);
      }
    },
    s02Key(newValue) {
      if (newValue == true) {
        console.log("Turning s02Key on: ", newValue);
      } else {
        console.log("Turning s02Key off: ", newValue);
      }
    },
    s03Key(newValue) {
      if (newValue == true) {
        console.log("Turning s03Key on: ", newValue);
      } else {
        console.log("Turning s03Key off: ", newValue);
      }
    },
    PGMlayouts(newValue) {
      console.log("PGM Layout Changed to: ", newValue);
    },
  },
  computed: {},
  created() {
    console.log("Main Dashboard created for event: ", this.eventKey);
    this.loadStrapFonts();
    // dummy live and rec icons
    setTimeout(() => {
      this.liveIcon = true;
      this.recordIcon = true;
    }, 5000);
  },
  methods: {
    mainCamera(mainCam, divID) {
      console.log("setting camera: ", mainCam, " as main camera");
      // making main camera border red
      let camsBorder = document.getElementsByClassName("live-video");
      camsBorder.forEach((ele) => {
        ele.style.cssText = "border-color: #666;";
      });
      document.getElementById("liveCam" + divID).style.cssText =
        "border-color: red;";
      // sets main camera as CAM1 for all layouts
      this.layouts.forEach((element) => {
        element.cams[0].camSelect = mainCam;
      });
      this.layoutSet();
    },
    bgUpload(e) {
      // upload background image
      var files = e;
      if (!files.length) {
        console.log("Trying to upload background image name: ", files.name);
        this.bgUploadIcon = true;
        return;
      }
    },
    logoUpload(e) {
      // upload logo image
      var files = e;
      if (!files.length) {
        console.log("Trying to upload logo image: ", files.name);
        this.logoUploadIcon = true;
        return;
      }
    },
    bugUpload(e) {
      // upload bug image
      var files = e;
      if (!files.length) {
        console.log("Trying to upload bug image: ", files.name);
        this.bugUploadIcon = true;
        return;
      }
    },
    loadStrapFonts() {
      this.tempFonts.forEach((element) => {
        this.strapFonts.push(element);
      });
    },
    strapUpdate() {
      this.straps.forEach((strap) => {
        if (strap.textArea.length > 0) {
          strap.liveReady = true;
        }
        if (strap.textArea.length == 0) {
          strap.liveReady = false;
          if (strap.id == 1) {
            this.s01Key = false;
          }
          if (strap.id == 2) {
            this.s02Key = false;
          }
          if (strap.id == 3) {
            this.s03Key = false;
          }
        }
      });
      console.log("Trying to update a Strapes: ", this.straps);
    },
    layoutSet() {
      this.layouts.forEach((element) => {
        console.log("layout: ", element.title);
        element.cams.forEach((element) => {
          console.log(element.camLabel, ": ", element.camSelect);
        });
      });
    },
    initJanus() {
      this.loading = true;
      // test server
      let server = "https://janus.conf.meetecho.com/janus";
      // vmcloudserver
      // let server = "wss://hlsdvr.vmclouds.co.uk:8989";
      Janus.init({
        // debug: true,
        dependencies: Janus.useDefaultDependencies(),
        callback: () => {
          this.connect(server);
        },
        error: (cause) => {
          Janus.error(cause);
          console.log("Server might be down, trying to reload!");
          setTimeout(window.location.reload(), 5000);
        },
        destroyed: () => {
          Janus.log("janus init destroyed");
        },
      });
    },
    // helper function to connect to Janus server and get streams list
    connect(server) {
      this.janus = new Janus({
        server,
        success: () => {
          this.loading = false;
        },
        error: (error) => {
          Janus.error("Failed to connect to janus server", error);
        },
        destroyed: () => {
          window.location.reload();
        },
      });
    },
    initJanusPGM() {
      this.loadingPGM = true;
      // public test server
      let server = "wss://janus-legacy.conf.meetecho.com/ws";
      
      // old public test server
      // let server = "https://janus.conf.meetecho.com/janus";
      
      // vmclouds server
      // let server = "wss://hlsdvr.vmclouds.co.uk:8989";
      Janus.init({
        // debug: true,
        dependencies: Janus.useDefaultDependencies(),
        callback: () => {
          this.connectPGM(server);
        },
        error: (cause) => {
          Janus.error(cause);
          console.log("Server might be down, trying to reload!");
          setTimeout(window.location.reload(), 5000);
        },
        destroyed: () => {
          Janus.log("janus init destroyed");
        },
      });
    },
    // helper function to connect to Janus server and get streams list
    connectPGM(server) {
      this.janusPGM = new Janus({
        server,
        success: () => {
          this.loadingPGM = false;
        },
        error: (error) => {
          Janus.error("Failed to connect to janus server", error);
        },
        destroyed: () => {
          window.location.reload();
        },
      });
    },
  },
};
</script>

<style>
#livePGMContianer {
  margin: auto;
  /* width: 80vw; */
  height: 60vh;
  overflow: hidden;
}
/* works with Janus component */
#livePGMContianer div div div video {
  /* Make video to at least 100% wide and tall */
  min-width: 96%;
  min-height: auto;

  /* Setting width & height to auto prevents the browser from stretching or squishing the video */
  /* width: auto; */
  /* height: auto; */

  /* Center the video */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
#prvAreaJanus {
  height: 250px;
  display: flex;
  overflow-x: auto;
  padding-bottom: 5vh;
}
#prvAreaJanus div div {
  display: flex !important;
  /* margin-left: 2vw; */
  /* cursor: move; */
}
/* #prvAreaJanus div{
  	display: flex !important;
} */
.prvFeedDrag {
  border: 3px solid #666;
  border-radius: 0.5em;
}
.rightMenu {
  background-color: black;
  color: white;
}
/* .rightMenu-active{
background-color: white;
color:black;
} */
.rightMenuContainer {
  overflow-y: auto;
}
.v-list-group--active {
  background-color: grey;
  color: black !important;
}
.mdi-close-thick {
  color: red !important;
}
.mdi-check-bold {
  color: green !important;
}
.strapTextArea textarea {
  color: #000 !important;
}
.fontSelect * {
  color: #000 !important;
}
.fontSelect div {
  color: #000 !important;
}
.v-menu__content div {
  color: #000 !important;
}
</style>
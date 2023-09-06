# WebMix 
This is a Frontend Prototype for Web App simulate video mixer functions in traitional gallery rooms.

To start using this repo run: 
npm i

Then:
npm run serve

When it finish you will see url with port number to visit the web page: usually localhost:8080 or 127.0.0.1:8080 but it may change the port number if it was already used.

To gain access to the dashboard you can use the follwoing link:
http://localhost:8080/webmix/123-test-123

This dashboard is generated from 3 files only, if you need to use to change or copy it:
src/views/WebMix.vue
src/views/JanusVideo.vue
src/views/JanusVideoPGM.vue

The app is built on Vue framework, each file contains HTML, JS and CSS.
Mostly if you will integrate Backend service with it you will work only in the JS parts in each file, there are already some dummy functions with console logs feedback for any built functions.
The video feed is coming from: https://janus.conf.meetecho.com/janus If you have your own Janus server you can use it directly.

*The project was built from about a year and half ago without proper updates, I think main core library that handles the video feeds "Janus WebRTC" was updated and the code still use the legacy version of it which may cause some issues if you have to update.

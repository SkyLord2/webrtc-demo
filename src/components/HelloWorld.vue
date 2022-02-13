<template>
    <div class="hello">
        <div @click="play" class="btn">播放视频</div>
        <video id="rtc_media_player" controls autoplay></video>
        <div><span>Session id:</span><span>{{sessionid}}</span></div>
        <div><span>Simulator:</span><span>{{simulator}}</span></div>
    </div>
</template>

<script>
import { SrsRtcPlayerAsync } from '../libs/srs.sdk';
export default {
    name: 'HelloWorld',
    data() {
        return {
            sdk: null,
            sessionid: '',
            simulator: ''
        };
    },
    props: {
        msg: String
    },
    methods: {
        play() {
            var video = document.getElementById('rtc_media_player');
            // Close PC when user replay.
            if (this.sdk) {
                this.sdk.close();
            }
            this.sdk = new SrsRtcPlayerAsync();

            // https://webrtc.org/getting-started/remote-streams
            video.srcObject = this.sdk.stream;
            // Optional callback, SDK will add track to stream.
            // sdk.ontrack = function (event) { console.log('Got track', event); sdk.stream.addTrack(event.track); };

            // For example: webrtc://r.ossrs.net/live/livestream
            var url = 'webrtc://124.222.18.231:80/live/livestream';
            this.sdk.play(url).then((session) => {
                this.sessionid = session.sessionid;
                this.simulator = session.simulator;
            }).catch((reason) => {
                this.sdk.close();
                console.error(reason);
            });
        },
        stop() {
            var video = document.getElementById('rtc_media_player');
            video.srcObject = null;
            video.pause();
        }
    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.btn {
    width: 70px;
    border: 1px solid #42b983;
    background-color: #42b983;
    cursor: pointer;
    margin: auto;
    margin-bottom: 20px;
}
</style>

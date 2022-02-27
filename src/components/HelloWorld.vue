<template>
    <div class="hello">
        <div @click="play" class="btn">播放视频</div>
        <video id="rtc_media_player" ref="rtcMediaPlayer" controls autoplay></video>
        <div><span>Session id:</span><span>{{sessionid}}</span></div>
        <div><span>Audio:</span><span>{{audioCodec}}</span></div>
        <div><span>Video:</span><span>{{videoCodec}}</span></div>
        <div><span>Simulator:</span><a :href="simulator">Drop</a></div>
    </div>
</template>

<script>
import { SrsRtcPlayerAsync, SrsRtcPublisherAsync, SrsRtcFormatSenders, SrsError } from '../libs/srs.sdk';
export default {
    name: 'HelloWorld',
    data() {
        return {
            sdk: null,
            sessionid: '',
            simulator: '',
            audioCodec: '',
            videoCodec: '',
        };
    },
    props: {
        msg: String
    },
    methods: {
        play() {
            var video = this.$refs.rtcMediaPlayer;
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
                this.simulator = session.simulator + '?drop=1&username=' + session.sessionid;
            }).catch((reason) => {
                this.sdk.close();
                console.error(reason);
            });
        },
        stop() {
            var video = this.$refs.rtcMediaPlayer;
            video.srcObject = null;
            video.pause();
        },
        publish() {
            var video = this.$refs.rtcMediaPlayer;
            // Close PC when user replay.
            if (this.sdk) {
                this.sdk.close();
            }
            this.sdk = new SrsRtcPublisherAsync();

            // User should set the stream when publish is done, @see https://webrtc.org/getting-started/media-devices
            // However SRS SDK provides a consist API like https://webrtc.org/getting-started/remote-streams
            video.srcObject = this.sdk.stream;
            // Optional callback, SDK will add track to stream.
            // sdk.ontrack = function (event) { console.log('Got track', event); sdk.stream.addTrack(event.track); };

            // https://developer.mozilla.org/en-US/docs/Web/Media/Formats/WebRTC_codecs#getting_the_supported_codecs
            this.sdk.pc.onicegatheringstatechange = function (event) {
                if (this.sdk.pc.iceGatheringState === "complete") {
                    this.audioCodec = SrsRtcFormatSenders(this.sdk.pc.getSenders(), "audio");
                    this.videoCodec = SrsRtcFormatSenders(this.sdk.pc.getSenders(), "video");
                }
            };

            // For example: webrtc://r.ossrs.net/live/livestream "webrtc://124.222.18.231/live/livestream.flv"
            var url = 'webrtc://124.222.18.231/live/livestream.flv';
            this.sdk.publish(url).then(function(session){
                this.sessionid = session.sessionid;
                this.simulator = session.simulator + '?drop=1&username=' + session.sessionid;
            }).catch(function (reason) {
                // Throw by sdk.
                if (reason instanceof SrsError) {
                    if (reason.name === 'HttpsRequiredError') {
                        alert(`WebRTC推流必须是HTTPS或者localhost：${reason.name} ${reason.message}`);
                    } else {
                        alert(`${reason.name} ${reason.message}`);
                    }
                }
                // See https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia#exceptions
                if (reason instanceof DOMException) {
                    if (reason.name === 'NotFoundError') {
                        alert(`找不到麦克风和摄像头设备：getUserMedia ${reason.name} ${reason.message}`);
                    } else if (reason.name === 'NotAllowedError') {
                        alert(`你禁止了网页访问摄像头和麦克风：getUserMedia ${reason.name} ${reason.message}`);
                    } else if (['AbortError', 'NotAllowedError', 'NotFoundError', 'NotReadableError', 'OverconstrainedError', 'SecurityError', 'TypeError'].includes(reason.name)) {
                        alert(`getUserMedia ${reason.name} ${reason.message}`);
                    }
                }

                this.sdk.close();
                console.error(reason);
            });
        },
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

<template>

    <div clas="container">
        <video ref="previewVideo" autoplay muted playsinline></video>
    </div>
</template>

<script lang="ts">

    import {ZegoExpressEngine} from 'zego-express-engine-webrtc'
    import {Component, Vue} from "vue-property-decorator";

    const config = {
        appId: 873296761,
        serverUrl: 'wss://webliveroom-test.zego.im/ws',
        tokenUrl: 'https://wsliveroom-alpha.zego.im:8282/token'
    }

    const zg = new ZegoExpressEngine(config.appId, config.serverUrl);

    @Component
    export default class ZegoPlay extends Vue {

        public $refs!: {
            previewVideo: HTMLVideoElement
        }

        constructor() {
            super();

            zg.on('roomStateUpdate', (roomID, state, errorCode, extendData) => {
                console.log('更新房间状态：' + roomID);
            });

            zg.on('roomStreamUpdate', (roomID, state, streamList) => {
                console.log('更新房间状态：' + roomID, state, streamList);

                if (streamList && streamList.length > 0) {

                    streamList.forEach(item => {
                        const {streamID} = item;

                        zg.startPlayingStream(streamID).then(pullStream => {
                            this.$refs.previewVideo.srcObject = pullStream;
                        })

                    })
                }
            })

            const userID = "player" + new Date().getTime();

            fetch(config.tokenUrl + "?app_id=" + config.appId + "&id_name=" + userID, {
                method: 'GET'
            }).then(res => {
                res.text().then(token => {

                    //登录房间
                    zg.loginRoom('10025', token, {userID, userName: 'peter'})
                        .then(loginSuccess => {

                            if (loginSuccess) {

                                // zg.startPlayingStream('20190609000').then(pullStream => {
                                //
                                //     this.$refs.previewVideo.srcObject = pullStream;
                                // }).catch(err => {
                                //     alert('拉流失败');
                                // })
                            }
                        }).catch(err => {
                        alert("登录房间失败");
                    })
                })
            }).catch(err => {
                console.log('获取token失败');
            })
        }
    }
</script>

<style scoped>

</style>
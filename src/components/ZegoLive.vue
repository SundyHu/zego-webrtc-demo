<template>

    <div class="container">

        <div>
            <label>音频</label>
            <select>
                <option v-for="item in videoDeviceList" :key="item.deviceID" v-bind:value="item.deviceID">
                    {{item.deviceName}}
                </option>
            </select>
        </div>

        <div class="previewVideo">
            <video ref="lv" autoplay muted></video>
        </div>
    </div>
</template>

<script lang="ts">

    import {ZegoExpressEngine} from 'zego-express-engine-webrtc'

    import {Vue, Component, Ref} from 'vue-property-decorator'

    const config = {
        appId: 873296761,
        serverUrl: 'wss://webliveroom-test.zego.im/ws',
        tokenUrl: 'https://wsliveroom-alpha.zego.im:8282/token'
    }

    const zg = new ZegoExpressEngine(config.appId, config.serverUrl);

    @Component
    export default class ZegoLive extends Vue {

        public $refs!: {
            lv: HTMLVideoElement
        }
        videoDeviceList: any[] = [{
            deviceID: '11111',
            deviceName: 'test'
        }];


        constructor() {
            super();

            // zg.enumDevices().then(res => {
            //     const {microphones, cameras, speakers} = res;
            //     this.videoDeviceList = cameras;
            // })

            const userID = "live" + new Date().getTime();

            fetch(config.tokenUrl + "?app_id=" + config.appId + "&id_name=" + userID, {
                method: 'GET'
            }).then(res => {
                res.text().then(token => {
                    //登录房间

                    zg.loginRoom('10025', token, {userID, userName: 'james'})
                        .then(loginSuccess => {

                            if (loginSuccess) {
                                //创建流

                                zg.createStream().then(localStream => {

                                    //alert(this.$refs.lv)
                                    //this.streamList = [{localStream}];
                                    //alert(this.$refs.lv)
                                    this.$refs.lv.srcObject = localStream;

                                    zg.startPublishingStream('20190609000', localStream)
                                })
                            }
                        }).catch(err => {
                        alert('登录房间失败');
                    })
                })
            }).catch(err => {
                alert('获取Token失败');
            })
        }
    }
</script>

<style scoped>

</style>
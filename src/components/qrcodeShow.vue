<template>
  <div class="container">
    <!-- 上半部分 -->
    <div class="section" v-for="hIndex in highQuantity" :key="hIndex">
      <div class="qrcodeBox" v-for="wIndex in widthQuantity" :key="wIndex" :style="`width: ${qrcodeSize}px; height: ${qrcodeSize}px`">
        <img v-show="qrcodeExit" alt="Image" ref="qrCodeDiv" class="vue-qrcode" src=""/>
      </div>
    </div>
  </div>
</template>

<script>
import config from '@/config.js';

export default {

  data() {
    return {
      imgList: [
        '/img/code/0.jpg',
        '/img/code/1.jpg',
        '/img/code/2.jpg',
        '/img/code/3.jpg',
        '/img/code/4.jpg',
        '/img/code/5.jpg',
        '/img/code/6.jpg',
        '/img/code/7.jpg',
        '/img/code/8.jpg',
      ],
      query: {},
      qrcodeSize: 300,
      highQuantity: 2,
      widthQuantity: 2,
      imgDiv: [],
      socket: null,
      qrcodeExit: false,
      content1: "",
      qNo: "Q000000",
      qText: "暂无问题",
      qrCode: null,
      qrcodeNo: null,
      questionClipsList: [],
      currentIndex: 0,
      codeNum: this.highQuantity * this.widthQuantity,
    }
  },

  async mounted() {
    this.socket = new WebSocket(config.WebSocket_URL);

    // 监听WebSocket消息事件
    this.socket.onmessage = this.onMessage;

    this.startLoopShowQRCode(this.highQuantity * this.widthQuantity)
  },
  unmounted() {
    this.socket.close()
  },

  created() {
    const size = this.getQueryParam('qrcodeSize')
    const high = this.getQueryParam('highQuantity')
    const width = this.getQueryParam('widthQuantity')
    this.qrcodeSize = size ? size : this.qrcodeSize
    this.highQuantity = high ? parseInt(high) : this.highQuantity
    this.widthQuantity = width ? parseInt(width) : this.widthQuantity
  },
  methods: {
    getQueryParam(key) {
      const queryParams = new URLSearchParams(window.location.search);
      return queryParams.get(key);
    },
    /**
     * 开始轮巡播放二维码
     * @param codeNum
     * @returns {*[]}
     */
    startLoopShowQRCode(codeNum){
      this.qrcodeExit = true;
      this.intervalList = [];
      const dataList = this.chunkArray(this.imgList, codeNum);

      for(let num = 0; num < dataList.length; num ++){
        const intervalId  = this.LoopShowQRCode(this.$refs.qrCodeDiv[num], dataList[num]);
        this.intervalList.push(intervalId);
      }
    },

    endLoopShowQRCode(){
      this.qrcodeExit = false;
      for(const item of this.intervalList){
        clearInterval(item);
      }
    },

    LoopShowQRCode(el, dataList){
      let index = 0;

      return setInterval(() => {
        index = this.showQRCode(el, dataList, index)
      }, 100)
    },

    showQRCode(el, dataList, currentIndex) {
      if(dataList.length === currentIndex) currentIndex = 0
      el.src = dataList[currentIndex]
      currentIndex += 1
      return currentIndex
    },

    chunkArray(array, numberOfChunks) {
      const result = [];
      const length = array.length;
      const chunkSize = Math.ceil(length / numberOfChunks);
      let index = 0;

      while (index < length) {
        const chunk = array.slice(index, index + chunkSize);
        result.push(chunk);
        index += chunkSize;
      }

      return result;
    },

    onMessage(data) {
      const JSONData = JSON.parse(data.data);

      if (JSONData['method'] === 'add'){
        for(const item of JSONData['list']) {
          const pathString = `${config.QRCode_URL}${item}.jpg`
          this.imgList.push(pathString);
        }

        this.endLoopShowQRCode();
        this.startLoopShowQRCode(this.highQuantity * this.widthQuantity);

      }
    },
  }
}
</script>

<style>
.container {
  min-height: 90vh;
  display: flex;
  //justify-content: space-between;
  margin: 2rem;
}

.section {
  margin-right: 100px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.qrcodeBox {
  position: relative;
  border: 1px solid #ccc;
}

.vue-qrcode {
  object-fit: cover;
  width: 100%;
  height: 100%;
}

</style>

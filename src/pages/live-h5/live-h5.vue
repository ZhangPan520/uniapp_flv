<template>
  <view class="page">
    <!-- 方案1：H5 页面加返回按钮，通过 postMessage 与 App 通信实现返回 -->
    <!-- vue 页面下的 web-view 官方文档明确写着"默认充满屏幕不可控制大小"，
         所以返回按钮/播放按钮/全屏按钮全部在 HTML 内部用普通网页元素自绘，
         点击返回时通过 uni.postMessage 通知这个 vue 页面执行 uni.navigateBack() -->
    <web-view
      v-if="webviewSrc"
      class="webview"
      :src="webviewSrc"
      @message="onWebviewMessage"
    ></web-view>
    <view v-else class="fallback">
      <text>当前平台暂不支持 web-view 直播播放，请在 App/Android 端运行</text>
    </view>
  </view>
</template>

<script>
// 直播流地址：拿到具体 FLV 地址后，只需要改这里
const FLV_URL = 'https://srs.unsj.edu.ar/live/xama/xama.flv'

export default {
  data() {
    return {
      webviewSrc: '',
    }
  },
  onLoad() {
    const sysInfo = uni.getSystemInfoSync()
    const statusBarHeight = sysInfo.statusBarHeight || 0

    // #ifdef APP-PLUS
    this.webviewSrc =
      '/hybrid/html/flv-player-h5.html?url=' +
      encodeURIComponent(FLV_URL) +
      '&statusBarHeight=' +
      statusBarHeight
    // #endif

    // #ifndef APP-PLUS
    this.webviewSrc = ''
    // #endif
  },
  methods: {
    onWebviewMessage(event) {
      // 接收 flv-player-h5.html 内部通过 uni.postMessage 上报的事件
      const data = event.detail && event.detail.data
      const msg = Array.isArray(data) ? data[0] : data
      console.log('[web-view message]', msg)
      if (!msg) return
      if (msg.type === 'back') {
        uni.navigateBack({ delta: 1 })
      }
    },
  },
}
</script>

<style>
.page {
  width: 100%;
  height: 100%;
  background: #000;
}

.webview {
  width: 100%;
  height: 100%;
}

.fallback {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 40rpx;
  text-align: center;
  color: #8f8f94;
  background: #000;
}
</style>

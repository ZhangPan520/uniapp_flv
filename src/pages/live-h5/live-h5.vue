<template>
  <view class="page">
    <!-- 方案1：H5 页面加返回按钮，通过 App WebView 桥接实现返回 -->
    <!-- vue 页面下的 web-view 官方文档明确写着"默认充满屏幕不可控制大小"，
         所以返回按钮/播放按钮/全屏按钮全部在 HTML 内部用普通网页元素自绘，
         返回按钮在内嵌页先恢复竖屏，再通过 WebView API 返回上一页 -->
    <web-view
      v-if="webviewSrc"
      ref="liveWebview"
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
const PLAYER_UI_VERSION = '20260814-3'
const LANDSCAPE_GAMMA_THRESHOLD = 55
const PORTRAIT_GAMMA_THRESHOLD = 30
const PORTRAIT_BETA_THRESHOLD = 45

export default {
  data() {
    return {
      webviewSrc: '',
      isFullscreen: false,
      orientationWatchId: null,
      physicalOrientation: 'portrait',
      suppressAutoFullscreenUntilPortrait: false,
      isLeaving: false,
    }
  },
  onLoad() {
    const sysInfo = uni.getSystemInfoSync()
    const statusBarHeight = sysInfo.statusBarHeight || 0

    // #ifdef APP-PLUS
    plus.screen.lockOrientation('portrait-primary')
    plus.navigator.setFullscreen(false)
    this.startOrientationWatcher()
    this.webviewSrc =
      '/hybrid/html/flv-player-h5.html?url=' +
      encodeURIComponent(FLV_URL) +
      '&statusBarHeight=' +
      statusBarHeight +
      '&uiVersion=' +
      PLAYER_UI_VERSION
    // #endif

    // #ifndef APP-PLUS
    this.webviewSrc = ''
    // #endif
  },
  onUnload() {
    this.stopOrientationWatcher()
    // #ifdef APP-PLUS
    plus.screen.lockOrientation('portrait-primary')
    plus.navigator.setFullscreen(false)
    // #endif
  },
  onBackPress() {
    if (this.isFullscreen) {
      this.exitFullscreen(true)
      return true
    }
    this.restorePortraitMode()
    return false
  },
  methods: {
    restorePortraitMode() {
      // #ifdef APP-PLUS
      plus.screen.lockOrientation('portrait-primary')
      plus.navigator.setFullscreen(false)
      // #endif
    },
    startOrientationWatcher() {
      // #ifdef APP-PLUS
      if (this.orientationWatchId !== null || !plus.orientation) return
      this.orientationWatchId = plus.orientation.watchOrientation(
        rotation => this.handlePhysicalOrientation(rotation),
        error => console.warn('[live-h5 orientation]', error),
        { frequency: 250 }
      )
      // #endif
    },
    stopOrientationWatcher() {
      // #ifdef APP-PLUS
      if (this.orientationWatchId !== null && plus.orientation) {
        plus.orientation.clearWatch(this.orientationWatchId)
        this.orientationWatchId = null
      }
      // #endif
    },
    handlePhysicalOrientation(rotation) {
      if (!rotation || this.isLeaving) return
      const gamma = Math.abs(Number(rotation.gamma))
      const beta = Math.abs(Number(rotation.beta))
      if (!Number.isFinite(gamma) || !Number.isFinite(beta)) return

      if (gamma >= LANDSCAPE_GAMMA_THRESHOLD && beta < PORTRAIT_BETA_THRESHOLD) {
        this.physicalOrientation = 'landscape'
        // #ifdef APP-PLUS
        if (this.isFullscreen && !plus.navigator.isFullscreen()) {
          this.isFullscreen = false
          this.suppressAutoFullscreenUntilPortrait = true
        }
        // #endif
        if (!this.isFullscreen && !this.suppressAutoFullscreenUntilPortrait) this.enterFullscreen()
      } else if (gamma <= PORTRAIT_GAMMA_THRESHOLD && beta >= PORTRAIT_BETA_THRESHOLD) {
        this.physicalOrientation = 'portrait'
        this.suppressAutoFullscreenUntilPortrait = false
        // #ifdef APP-PLUS
        if (plus.navigator.isFullscreen()) this.isFullscreen = true
        // #endif
      }
    },
    evalInWebview(action) {
      const script = "window.dispatchEvent(new CustomEvent('uni-app-command', { detail: { action: '" + action + "' } }))"
      if (this.$refs.liveWebview && typeof this.$refs.liveWebview.evalJS === 'function') {
        this.$refs.liveWebview.evalJS(script)
        return
      }
      // #ifdef APP-PLUS
      const pageWebview = this.$scope && typeof this.$scope.$getAppWebview === 'function'
        ? this.$scope.$getAppWebview()
        : plus.webview.currentWebview()
      const children = pageWebview && pageWebview.children ? pageWebview.children() : []
      const playerWebview = children && children.find(child => {
        const url = typeof child.getURL === 'function' ? child.getURL() : ''
        return url.indexOf('flv-player-h5.html') !== -1
      })
      if (playerWebview) playerWebview.evalJS(script)
      // #endif
    },
    enterFullscreen() {
      if (this.isFullscreen || this.isLeaving) return
      this.isFullscreen = true
      this.evalInWebview('fullscreen_enter')
      // #ifdef APP-PLUS
      plus.screen.unlockOrientation()
      plus.navigator.setFullscreen(true)
      // #endif
    },
    exitFullscreen(suppressAutoEnter) {
      this.isFullscreen = false
      this.suppressAutoFullscreenUntilPortrait = Boolean(suppressAutoEnter)
      this.evalInWebview('fullscreen_exit')
      this.restorePortraitMode()
    },
    restorePortraitAndBack() {
      if (this.isLeaving) return
      this.isLeaving = true
      this.stopOrientationWatcher()
      this.isFullscreen = false
      this.evalInWebview('fullscreen_exit')
      this.restorePortraitMode()
      setTimeout(() => uni.navigateBack({ delta: 1 }), 80)
    },
    extractMessage(event) {
      let msg = event && event.detail && event.detail.data !== undefined ? event.detail.data : event
      for (let depth = 0; depth < 6 && msg && !msg.type; depth += 1) {
        if (Array.isArray(msg)) msg = msg[0]
        else if (msg.detail !== undefined) msg = msg.detail
        else if (msg.data !== undefined) msg = msg.data
        else break
      }
      return msg
    },
    onWebviewMessage(event) {
      // 接收 flv-player-h5.html 内部通过 uni.postMessage 上报的事件
      const msg = this.extractMessage(event)
      console.log('[web-view message]', msg)
      if (!msg) return
      if (msg.type === 'ready') this.evalInWebview(this.isFullscreen ? 'fullscreen_enter' : 'fullscreen_exit')
      else if (msg.type === 'request_back' || msg.type === 'back') this.restorePortraitAndBack()
      else if (msg.type === 'request_fullscreen_enter' || msg.type === 'fullscreen_enter') this.enterFullscreen()
      else if (msg.type === 'request_fullscreen_exit' || msg.type === 'fullscreen_exit') this.exitFullscreen(true)
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

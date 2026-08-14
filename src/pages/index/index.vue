<template>
  <view class="page">
    <view class="status-space" :style="{ height: statusBarHeight + 'px' }"></view>

    <view class="header">
      <view>
        <text class="eyebrow">滨江公寓</text>
        <text class="title">我的设备</text>
      </view>
      <view class="add-button" @click="showToast('添加设备入口')">
        <text class="add-icon">+</text>
      </view>
    </view>

    <scroll-view class="device-list" scroll-y>
      <view class="device-card" @click="openLivePicker">
        <image class="preview" src="/static/monitor-office.jpg" mode="aspectFill"></image>
        <view class="live-mark"><view class="live-dot"></view><text class="live-text">实时</text></view>
        <view class="device-copy">
          <view class="device-main">
            <text class="device-name">客厅摄像头</text>
            <view class="device-state"><view class="online-dot"></view><text class="device-detail">在线 · 刚刚更新</text></view>
          </view>
          <text class="chevron">›</text>
        </view>
      </view>

      <view class="page-note">
        <text class="page-note-title">1 台设备在线</text>
        <text class="page-note-copy">画面连接稳定</text>
      </view>
    </scroll-view>

    <view v-if="pickerVisible" class="sheet-mask" @click="pickerVisible = false">
      <view class="sheet" @click.stop>
        <view class="sheet-handle"></view>
        <text class="sheet-title">选择播放方案</text>
        <text class="sheet-copy">两种方案使用相同的监控界面与直播源</text>
        <view class="sheet-option" @click="goTo('/pages/live-h5/live-h5')">
          <view><text class="option-title">H5 自绘 UI</text><text class="option-copy">完整界面由 WebView 渲染</text></view>
          <text class="option-arrow">›</text>
        </view>
        <view class="sheet-option" @click="goTo('/pages/live-nvue/live-nvue')">
          <view><text class="option-title">nvue 原生页面</text><text class="option-copy">原生布局承载播放器</text></view>
          <text class="option-arrow">›</text>
        </view>
        <view class="sheet-cancel" @click="pickerVisible = false"><text class="cancel-text">取消</text></view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      statusBarHeight: 0,
      pickerVisible: false,
    }
  },
  onLoad() {
    const systemInfo = uni.getSystemInfoSync()
    this.statusBarHeight = systemInfo.statusBarHeight || 0
  },
  methods: {
    openLivePicker() {
      this.pickerVisible = true
    },
    goTo(url) {
      this.pickerVisible = false
      uni.navigateTo({ url })
    },
    showToast(title) {
      uni.showToast({ title, icon: 'none' })
    },
  },
}
</script>

<style>
page { background: #f5f7f5; }
.page { min-height: 100vh; background: #f5f7f5; color: #161818; }
.status-space { background: #f5f7f5; }
.header { display: flex; align-items: flex-end; justify-content: space-between; padding: 28rpx 40rpx 20rpx; }
.eyebrow { display: block; margin-bottom: 4rpx; color: #737977; font-size: 22rpx; line-height: 32rpx; font-weight: 600; }
.title { display: block; color: #161818; font-size: 56rpx; line-height: 72rpx; font-weight: 700; }
.add-button { width: 88rpx; height: 88rpx; display: flex; align-items: center; justify-content: center; }
.add-icon { color: #161818; font-size: 48rpx; line-height: 48rpx; font-weight: 300; }
.device-list { box-sizing: border-box; height: calc(100vh - 190rpx); padding: 20rpx 28rpx 56rpx; }
.device-card { position: relative; overflow: hidden; background: #fff; border: 2rpx solid #e5e8e6; border-radius: 14rpx; }
.preview { display: block; width: 100%; height: 392rpx; background: #111413; filter: saturate(.78) brightness(.82); }
.live-mark { position: absolute; top: 24rpx; left: 24rpx; min-height: 52rpx; padding: 0 16rpx; display: flex; align-items: center; background: rgba(12, 15, 14, .66); border: 2rpx solid rgba(255,255,255,.14); border-radius: 10rpx; }
.live-dot { width: 12rpx; height: 12rpx; margin-right: 12rpx; background: #e54936; border-radius: 50%; }
.live-text { color: #fff; font-size: 22rpx; line-height: 32rpx; font-weight: 700; }
.device-copy { min-height: 122rpx; padding: 24rpx 28rpx; display: flex; align-items: center; justify-content: space-between; box-sizing: border-box; }
.device-main { min-width: 0; }
.device-name { display: block; color: #161818; font-size: 30rpx; line-height: 42rpx; font-weight: 700; }
.device-state { margin-top: 4rpx; display: flex; align-items: center; }
.online-dot { width: 12rpx; height: 12rpx; margin-right: 10rpx; background: #28a06a; border-radius: 50%; }
.device-detail { color: #737977; font-size: 22rpx; line-height: 32rpx; }
.chevron { flex: none; color: #848a87; font-size: 44rpx; font-weight: 300; }
.page-note { padding: 32rpx 4rpx; }
.page-note-title, .page-note-copy { display: block; color: #737977; font-size: 22rpx; line-height: 34rpx; }
.page-note-title { color: #454a48; font-weight: 700; }
.sheet-mask { position: fixed; inset: 0; z-index: 20; display: flex; align-items: flex-end; background: rgba(10, 12, 11, .42); }
.sheet { width: 100%; padding: 16rpx 28rpx calc(24rpx + env(safe-area-inset-bottom)); box-sizing: border-box; background: #f5f7f5; border-radius: 16rpx 16rpx 0 0; }
.sheet-handle { width: 64rpx; height: 8rpx; margin: 0 auto 24rpx; background: #c7ccc9; border-radius: 4rpx; }
.sheet-title { display: block; padding: 0 8rpx; color: #161818; font-size: 32rpx; line-height: 46rpx; font-weight: 700; }
.sheet-copy { display: block; padding: 2rpx 8rpx 22rpx; color: #737977; font-size: 22rpx; line-height: 34rpx; }
.sheet-option { min-height: 112rpx; margin-bottom: 12rpx; padding: 20rpx 24rpx; display: flex; align-items: center; justify-content: space-between; box-sizing: border-box; background: #fff; border: 2rpx solid #e5e8e6; border-radius: 12rpx; }
.option-title, .option-copy { display: block; }
.option-title { color: #161818; font-size: 28rpx; line-height: 40rpx; font-weight: 700; }
.option-copy { color: #737977; font-size: 21rpx; line-height: 32rpx; }
.option-arrow { color: #848a87; font-size: 42rpx; }
.sheet-cancel { height: 88rpx; display: flex; align-items: center; justify-content: center; }
.cancel-text { color: #454a48; font-size: 27rpx; font-weight: 600; }
</style>

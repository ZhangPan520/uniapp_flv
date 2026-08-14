<template>
  <view class="page">
    <!-- App 端优先使用同名 .nvue 页面；该文件仅为其他平台提供提示。 -->
    <view class="fallback">
      <text>当前平台暂不支持 web-view 直播播放，请在 App/Android 端运行</text>
    </view>
  </view>
</template>

<script>
export default {}
</script>

<style>
.page {
  width: 100%;
  height: 100%;
  background: #000;
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

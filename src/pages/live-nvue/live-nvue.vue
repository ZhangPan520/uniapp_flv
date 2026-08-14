<template>
  <view class="page">
    <!-- 该文件仅在 H5 / 小程序端生效：App 端存在同名 live.nvue，uni-app 会优先编译使用 .nvue -->
    <!-- App 端的三段式布局（导航栏 + web-view + 播放控制栏）在 live.nvue 中实现，
         原因是 vue 页面下的 web-view 官方文档明确写着"默认充满屏幕不可控制大小"，
         只有 nvue 页面的 web-view 才支持用 flex/宽高样式做局部布局 -->
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

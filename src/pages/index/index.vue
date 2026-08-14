<template>
  <view class="page">
    <view class="header">
      <text class="title">我的应用XXX</text>
    </view>

    <view class="grid">
      <view class="card" @click="goTo('/pages/live-h5/live-h5')">
        <view class="card-icon">▶</view>
        <text class="card-text">方案1：H5自绘UI</text>
      </view>
      <view class="card" @click="goTo('/pages/live-nvue/live-nvue')">
        <view class="card-icon">▶</view>
        <text class="card-text">方案2：nvue原生页面</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    goTo(url) {
      uni.navigateTo({ url })
    },
  },
}
</script>

<style>
.page {
  width: 100%;
  height: 100%;
  background: #f8f8f8;
}

.header {
  padding: 40rpx 30rpx 20rpx;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #333;
}

.grid {
  display: flex;
  flex-wrap: wrap;
  padding: 20rpx;
}

.card {
  width: 200rpx;
  height: 200rpx;
  background: #fff;
  border-radius: 16rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.06);
  margin: 10rpx;
  padding: 10rpx;
  box-sizing: border-box;
}

.card-icon {
  font-size: 48rpx;
  color: #007aff;
  margin-bottom: 12rpx;
}

.card-text {
  font-size: 22rpx;
  color: #333;
  text-align: center;
}
</style>

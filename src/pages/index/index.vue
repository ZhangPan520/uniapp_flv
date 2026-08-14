<template>
  <view class="page">
    <view class="header">
      <text class="title">我的应用XXX</text>
    </view>

    <view class="grid">
      <view class="card" @click="goLive">
        <view class="card-icon">▶</view>
        <text class="card-text">直播播放</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    goLive() {
      uni.navigateTo({
        url: '/pages/live/live',
      })
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
}

.card-icon {
  font-size: 48rpx;
  color: #007aff;
  margin-bottom: 12rpx;
}

.card-text {
  font-size: 26rpx;
  color: #333;
}
</style>

<template>
  <view ref="container">
    <image :src="`${$VUE_APP_RESOURCES_URL}/images/banner_coupon.png`" mode="widthFix" class="tui-coupon-banner"></image>
    <view class="tui-coupon-list">
      <view class="tui-coupon-item tui-top20" v-for="(item, index) in couponsList" :key="index">
        <image :src="`${$VUE_APP_RESOURCES_URL}/images/bg_coupon_3x.png`" class="tui-coupon-bg" mode="widthFix"></image>
        <view class="tui-coupon-item-left">
          <view class="tui-coupon-price-box" :class="{ 'tui-color-grey': item.isUse }">
            <view class="tui-coupon-price-sign">￥</view>
            <view class="tui-coupon-price" :class="{ 'tui-price-small': false }">{{ item.couponPrice }}</view>
          </view>
          <view class="tui-coupon-intro">满{{ item.useMinPrice }}元可用</view>
        </view>
        <view class="tui-coupon-item-right">
          <view class="tui-coupon-content">
            <view class="tui-coupon-title-box">
              <view class="tui-coupon-btn" v-if="item.ctype === 0" :class="{ 'tui-bg-grey': item.isUse }">通用劵</view>
              <view class="tui-coupon-btn" v-else-if="item.ctype === 1" :class="{ 'tui-bg-grey': item.isUse }">商品券</view>
              <view class="tui-coupon-btn" v-else :class="{ 'tui-bg-grey': item.isUse }">未知</view>
              <view class="tui-coupon-title">{{ item.cname }}</view>
            </view>
            <view class="tui-coupon-rule">
              <view class="tui-rule-box tui-padding-btm">
                <view class="tui-coupon-circle"></view>
                <view class="tui-coupon-text">不可叠加使用</view>
              </view>
              <view class="tui-rule-box">
                <view class="tui-coupon-circle"></view>
                <view class="tui-coupon-text" v-if="item.endTime !== 0">{{ item.endTime }} 到期</view>
                <!-- <view class="tui-coupon-text" v-if="item.endTime !== 0">{{ item.startTime }} - {{ item.endTime }}</view> -->
                <view class="tui-coupon-text" v-else>不限时</view>
              </view>
            </view>
          </view>
        </view>
        <view class="tui-btn-box">
          <tui-button type="danger" width="152rpx" height="52rpx" :size="24" shape="circle" :plain="true" v-if="item.isUse === true">已领取</tui-button>
          <tui-button type="danger" width="152rpx" height="52rpx" :size="24" shape="circle" :plain="true" v-else-if="item.isUse === 2">已领完</tui-button>
          <tui-button type="danger" width="152rpx" height="52rpx" :size="24" shape="circle" v-else @click="getCoupon(item.id, index)">立即领取</tui-button>
        </view>
      </view>
    </view>

    <Loading :loaded="loadend" :loading="loading"></Loading>
    <!--暂无优惠券-->
    <view class="noCommodity" v-if="couponsList.length === 0 && page > 1">
      <view class="noPictrue">
        <image :src="`${$VUE_APP_RESOURCES_URL}/images/noCoupon.png`" class="image" />
      </view>
    </view>
  </view>
</template>
<script>
import { getCoupon, getCouponReceive } from '@/api/user'
import Loading from '@/components/Loading'
import DataFormatT from '@/components/DataFormatT'
export default {
  name: 'getCoupon',
  components: {
    Loading,
    DataFormatT,
  },
  props: {},
  data: function() {
    return {
      page: 1,
      limit: 10,
      couponsList: [],
      loading: false,
      loadend: false,
    }
  },
  mounted: function() {
    this.getUseCoupons()
  },
  onReachBottom() {
    !this.loading && this.getUseCoupons()
  },
  methods: {
    getCoupon: function(id, index) {
      let that = this
      let list = that.couponsList
      getCouponReceive(id)
        .then(function(res) {
          list[index].isUse = true
          uni.showToast({
            title: '领取成功',
            icon: 'success',
            duration: 2000,
          })
        })
        .catch(function(err) {
          uni.showToast({
            title: err.msg || err.response.data.msg || err.response.data.message,
            icon: 'none',
            duration: 2000,
          })
        })
    },
    getUseCoupons() {
      if (this.loading) return //阻止下次请求（false可以进行请求）；
      if (this.loadend) return //阻止结束当前请求（false可以进行请求）；
      this.loading = true
      let q = { page: this.page, limit: this.limit }
      getCoupon(q).then(res => {
        this.loading = false
        //apply();js将一个数组插入另一个数组;
        this.couponsList.push.apply(this.couponsList, res.data)
        this.loadend = res.data.length < this.limit //判断所有数据是否加载完成；
        this.page = this.page + 1
      })
    },
  },
}
</script>

<style lang="less" scoped>
page {
  background-color: #f5f5f5;
}

.container {
  padding-bottom: env(safe-area-inset-bottom);
}

.tui-coupon-list {
  width: 100%;
  padding: 0 25rpx;
  box-sizing: border-box;
}

.tui-coupon-banner {
  width: 100%;
}

.tui-coupon-item {
  width: 100%;
  height: 210rpx;
  position: relative;
  display: flex;
  align-items: center;
  padding-right: 30rpx;
  box-sizing: border-box;
  overflow: hidden;
}

.tui-coupon-bg {
  width: 100%;
  height: 210rpx;
  position: absolute;
  left: 0;
  top: 0;
  z-index: 1;
}

.tui-coupon-sign {
  height: 110rpx;
  width: 110rpx;
  position: absolute;
  z-index: 9;
  top: -30rpx;
  right: 40rpx;
}

.tui-coupon-item-left {
  width: 218rpx;
  height: 210rpx;
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  flex-shrink: 0;
}

.tui-coupon-price-box {
  display: flex;
  color: #e41f19;
  align-items: flex-end;
}

.tui-coupon-price-sign {
  font-size: 30rpx;
}

.tui-coupon-price {
  font-size: 70rpx;
  line-height: 68rpx;
  font-weight: bold;
}

.tui-price-small {
  font-size: 58rpx !important;
  line-height: 56rpx !important;
}

.tui-coupon-intro {
  background: #f7f7f7;
  padding: 8rpx 10rpx;
  font-size: 26rpx;
  line-height: 26rpx;
  font-weight: 400;
  color: #666;
  margin-top: 18rpx;
}

.tui-coupon-item-right {
  flex: 1;
  height: 210rpx;
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-left: 24rpx;
  box-sizing: border-box;
  overflow: hidden;
}

.tui-coupon-content {
  width: 82%;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.tui-coupon-title-box {
  display: flex;
  align-items: center;
}

.tui-coupon-btn {
  padding: 6rpx;
  background: #ffebeb;
  color: #e41f19;
  font-size: 25rpx;
  line-height: 25rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: scale(0.9);
  transform-origin: 0 center;
  border-radius: 4rpx;
  flex-shrink: 0;
}

.tui-color-grey {
  color: #888 !important;
}

.tui-bg-grey {
  background: #f0f0f0 !important;
  color: #888 !important;
}

.tui-coupon-title {
  width: 100%;
  font-size: 26rpx;
  color: #333;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.tui-coupon-rule {
  padding-top: 52rpx;
}

.tui-rule-box {
  display: flex;
  align-items: center;
  transform: scale(0.8);
  transform-origin: 0 100%;
}

.tui-padding-btm {
  padding-bottom: 6rpx;
}

.tui-coupon-circle {
  width: 8rpx;
  height: 8rpx;
  background: rgb(160, 160, 160);
  border-radius: 50%;
}

.tui-coupon-text {
  font-size: 28rpx;
  line-height: 28rpx;
  font-weight: 400;
  color: #666;
  padding-left: 8rpx;
  white-space: nowrap;
}

.tui-top20 {
  margin-top: 20rpx;
}

.tui-coupon-title {
  font-size: 28rpx;
  line-height: 28rpx;
}

.tui-coupon-radio {
  transform: scale(0.7);
  transform-origin: 100% center;
}

.tui-btn-box {
  position: absolute;
  width: 146rpx;
  height: 52rpx;
  right: 20rpx;
  bottom: 40rpx;
  z-index: 10;
}
</style>

<!-- 货物详情 -->
<template>
  <view id="good-detail">
    <view class="good-list">
      <view class="card good" v-for="(name, idx) in nameList" :key="idx">
        <view class="good-title">
          <view class="index">
            {{ idx + 1 }}、{{ name }}
          </view>
        </view>
      </view>
    </view>

  </view>
</template>

<script>
import {localRead} from "../../lib/utils"

export default {
  data() {
    return {
      nameList:[]
    }
  },
  onLoad(){
    this.nameList = localRead('nameList') || []
  }
}
</script>

<style scoped lang="scss">
@import "../../static/styles/scss/geng.scss";

#good-detail {
  @include background-common;

  // 控件组
  .top-control-group {
    width: 100%;
    height: 120rpx;
    line-height: 120rpx;
    background: $bg-color-white;
    display: flex;
    align-items: center;
    padding-left: 20rpx;

    .input{
      width: calc(100% - 200rpx);
    }

    .btn{
      width: 180rpx;
      height: 76rpx;
      line-height: 70rpx;
      border-radius: 8rpx;
    }
  }

  // 商品列表
  .good-list {
    padding: $padding-common;

    .good {
      margin: 16rpx 0;

      .good-title {
        .index {
          margin-bottom: 8rpx;
          text-indent: 0;
        }

        .title {
          text-indent: 16rpx;
        }

        font-size: $font-size-small;
        color: $font-color-black;
      }
    }
  }
}
</style>

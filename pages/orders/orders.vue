<!-- 我的订单 -->
<template>
  <view id="orders">

    <!-- controls group -->
    <div class="control-group bg-white">
      <u-search v-model="form.orderId" :type="type" :border="true" focus="true" placeholder="请输入订单编号"
                @search="$u.throttle(queryPage, 500)" @custom="$u.throttle(queryPage, 500)"/>
    </div>

    <!-- card list -->
    <view class="card-list">
      <view class="card" v-for="(item, idx) in orderList" :key="idx"
            @click.self="toOrderDetail(item)">
        <view class="title">
          <view class="title-l">订单号: {{ item.id }}</view>
          <view class="title-r" @click="toOrderDetail(item)">查看详情 ></view>
        </view>
        <view class="place">
          <view class="place-l">
            <view class="item">
              <view class="ch-1 first" style="width: 280rpx;height: 120rpx;line-height: 120rpx">
                {{ item.senderArea | areaDisplay }}
              </view>
              <view class="ch-2">{{ item.sender || '未知寄件人' }}</view>
            </view>
          </view>
          <view class="place-c">
            <view class="item">
              <image style="width: 128rpx;height: 120rpx;" src="../../static/images/arrow.png"/>
              <view class="ch-2">{{ item.orderStatusName || '未知状态' }}</view>
            </view>
          </view>
          <view class="place-r">
            <view class="item">
              <view class="ch-1 second" style="width: 280rpx;height: 120rpx;line-height: 120rpx">
                {{ item.receiverArea | areaDisplay }}
              </view>
              <view class="ch-2">{{ item.receiver || '未知接收人' }}</view>
            </view>
          </view>
        </view>
        <view class="tip-1">最新物流：{{ item.lastFlowRecord.flowAddress || '暂无' }}</view>
        <view class="tip-2">更新时间：{{ item.lastFlowRecord.flowTime || '暂无' }}</view>
      </view>
      <view class="more">
        <u-loadmore icon-type="flower" :status="loadStatus" :load-text="loadText"/>
      </view>
    </view>

    <!-- 回到顶部 -->
    <view class="wrap">
      <u-back-top :scroll-top="scrollTop" mode="square" :icon-style="iconStyle"/>
    </view>

  </view>
</template>

<script>
import api from '../../api'
import {trimObj} from "../../lib/tools"
import containsCityName, {showLoading} from "../../lib/utils"

export default {
  data() {
    return {
      // 滚动条未知
      scrollTop: 0,
      // toTop 形状
      iconStyle: {
        color: '#FFF'
      },
      // 加载状态
      loadStatus: null,
      loadText: {
        loadmore: '轻轻上拉',
        loading: '努力加载中',
        nomore: '没有更多数据了'
      },
      form: {
        orderId: '',
        limit: 20,
        page: 1
      },
      orderList: []
    }
  },
  computed: {
    queryData() {
      let o = {...this.form}
      trimObj(o)

      return o
    }
  },
  filters: {
    areaDisplay(val) {
      if (!val) return ''

      let arr = val.split('/')
      for (const item of arr) {
        if (containsCityName(item)) {
          return item
        }
      }

      return ''
    }
  },
  methods: {
    // 跳转到订单详情
    toOrderDetail(item) {
      if (!item || !item.id) return -1
      uni.navigateTo({
        url: `../order-detail/order-detail?orderId=${item.id}`
      })
    },
    // 查询订单列表
    queryPage() {
      // loading animate
      showLoading()
      // query data
      api.listOrder(this.queryData).then(({data}) => {
        console.log(data)

        // stop loading animate
        uni.hideLoading()

        // update orderlsit
        this.orderList = data.records
      }).catch(err => {
        // stop loading animate
        uni.hideLoading()
      })
    },
    // 查询更多订单
    queryMorePage() {
      // play bottom animate
      this.loadStatus = 'loading'

      // modify query length
      this.form.limit += 20

      // query data
      api.listOrder(this.queryData).then(({data}) => {
        console.log(data)

        // stop animate
        this.loadStatus = null

        // roll back form modify
        if (data.records.length === this.orderList.length) {
          this.form.limit -= 20
        }

        // update orderlist
        this.orderList = data.records
      }).catch(err => {
        // stop loading animate
        uni.hideLoading()
      })
    }
  },
  onLoad() {
    this.queryPage()
  },
  // 下拉刷新事件
  onPullDownRefresh() {
    this.queryPage()
    uni.stopPullDownRefresh()
  },
  // 到达页面底部
  onReachBottom() {
    this.queryMorePage()
  },
  // 页面滚动条位移事件
  onPageScroll(e) {
    this.scrollTop = e.scrollTop;
  }
}
</script>

<style scoped lang="scss">
@import "../../static/styles/scss/geng.scss";

#orders {
  @include background-common;

  .bg-white {
    background: #FFFFFF;
  }


  .control-group {
    padding: 30rpx;
  }

  .card-list {
    padding: $padding-common;

    .card {
      padding: $padding-lg;

      // 订单号
      .title {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 32rpx;
        font-size: $font-size-small;
        color: $font-color-black;
      }

      // 起始地址
      .place {
        display: flex;
        flex-direction: row;
        justify-content: space-around;
        align-items: center;
        margin-bottom: 46rpx;
        color: $font-color-black;

        .item {
          display: flex;
          flex-direction: column;
          align-items: center;

          .ch-1 {
            font-size: 40rpx;
            font-weight: bold;
            text-align: center;

            &.first{
              color: $font-color-blue;
            }

            &.second{
              color: $font-color-green;
            }
          }

          .ch-2 {
            font-size: $font-size-small;
            color: $font-color-gray;
          }
        }
      }

      // 更新时间，最新物流
      .tip-2,
      .tip-1 {
        height: 40rpx;
        line-height: 40rpx;
        font-size: $font-size-small;
        overflow: hidden;
        color: #AAAAAA;
      }
    }
  }

}
</style>

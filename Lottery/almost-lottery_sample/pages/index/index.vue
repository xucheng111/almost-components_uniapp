<template>
  <view class="home">
    <view class="home-title">
      <text class="home-title__text">almost-lottery</text>
    </view>
    <view class="home-wheel">
      <almost-lottery
        :prize-list="prizeList"
        :ring-count="2"
        :duration="1"
        :canvas-width="canvasData.width"
        :canvas-height="canvasData.height"
        :prize-index="prizeIndex"
        @reset-index="prizeIndex = -1"
        @draw-start="handleDrawStart"
        @draw-end="handleDrawEnd"
        @finish="handleDrawFinish"
        v-if="prizeList.length"
      />
      <text class="almost-lottery__tip" v-else>奖品准备中...</text>
    </view>
    <view class="home-result">
      <text class="home-result__text">{{ targetName }}</text>
    </view>
    <view class="home-action" @tap="handleInitCanvas">
      <text class="home-action__text">重新生成-测试用</text>
    </view>
  </view>
</template>

<script>
  import AlmostLottery from '@/components/almost-lottery/almost-lottery.vue'
  export default {
    name: 'Home',
    components: {
      AlmostLottery
    },
    data () {
      return {
        // canvas 宽高
        canvasData: {
          width: 240,
          height: 240
        },
        // 奖品数据
        prizeList: [],
        // 中奖下标
        prizeIndex: -1,
        // 中奖类目名称
        targetName: '',
        // 是否由前端控制概率
        isFrontend: true,
        // 权重随机数的最大值
        weightTotal: 0,
        // 权重数组
        weightArr: []
      }
    },
    methods: {
      // 重新生成
      handleInitCanvas () {
        this.prizeList = []
        this.getPrizeList()
      },
      // 获取奖品列表
      getPrizeList () {
        uni.showLoading({
          title: '奖品准备中...'
        })
        // 模拟请求奖品列表
        let stoTimer = setTimeout(() => {
          clearTimeout(stoTimer)
          stoTimer = null
          
          // stock 奖品库存
          // weight 中奖概率，数值越大中奖概率越高
          this.prizeList = [
            { prizeId: 1, name: '0.1元现金', stock: 10, weight: 1, imgSrc: '/static/lottery-prize/git.png' },
            { prizeId: 2, name: '10元现金', stock: 0, weight: 0 },
            { prizeId: 3, name: '5元话费', stock: 1, weight: 0 },
            { prizeId: 4, name: '50元现金', stock: 0, weight: 0 },
            { prizeId: 5, name: '1卷抽纸', stock: 3, weight: 3 },
            { prizeId: 6, name: '0.2元现金', stock: 8, weight: 2 },
            { prizeId: 7, name: '谢谢参与', stock: 100, weight: 10000 },
            { prizeId: 8, name: '100金币', stock: 100, weight: 1000 }
          ]
          
          // 计算出权重的总和并生成权重数组
          if (this.isFrontend) {
            this.prizeList.forEach((item) => this.weightTotal += item.weight)
            this.weightArr = this.prizeList.map((item) => item.weight)
          }
        }, 500)
      },
      // 本次抽奖开始
      handleDrawStart () {
        this.targetName = ''
        
        let list = [...this.prizeList]
        
        // 模拟请求
        let stoTimer = setTimeout(() => {
          clearTimeout(stoTimer)
          stoTimer = null
          
          // 判断是否由前端控制概率
          // 前端控制概率的情况下，需要拿到最接近随机权重且大于随机权重的值
          // 后端控制概率的情况下，通常会直接返回 prizeId
          if (this.isFrontend && this.weightTotal) {
            console.warn('###当前处于前端控制中奖概率，安全起见，强烈建议由后端控制###')
            console.log('当前权重总和为 =>', this.weightTotal)
            
            // 注意这里使用了 Math.ceil，如果某个权重的值为 0，则始终无法中奖
            let weight = Math.ceil(Math.random() * this.weightTotal)
            console.log('本次权重随机数 =>', weight)
            
            // 生成大于等于随机权重的数组
            let tempMaxArrs = []
            list.forEach((item) => {
              if (item.weight >= weight) {
                tempMaxArrs.push(item.weight)
              }
            })
            
            // 如果大于随机权重的数组有值，先对这个数组排序然后取值
            // 反之新建一个临时的包含所有权重的已排序数组，然后取值
            if (tempMaxArrs.length) {
              tempMaxArrs.sort((a, b) => a - b)
              this.prizeIndex = this.weightArr.indexOf(tempMaxArrs[0])
            } else {
              let tempWeightArr = [...this.weightArr]
              tempWeightArr.sort((a, b) => a - b)
              this.prizeIndex = this.weightArr.indexOf(tempWeightArr[tempWeightArr.length - 1])
            }
          } else {
            // 这里随机产生的 prizeId 是模拟后端返回的 prizeId
            let prizeId = Math.floor(Math.random() * list.length)
            list.forEach((item, index) => {
              if (item.prizeId === prizeId) {
                // 中奖下标
                this.prizeIndex = index
              }
            })
          }
          
          console.log('本次抽中奖品 =>', this.prizeList[this.prizeIndex].name)
          console.log('本次奖品库存 =>', this.prizeList[this.prizeIndex].stock)
        }, 200)
      },
      // 本次抽奖结束
      handleDrawEnd () {
        // 旋转结束后，可以执行拿到结果后的逻辑
        let prizeName = this.prizeList[this.prizeIndex].name
        let prizeStock = this.prizeList[this.prizeIndex].stock
        this.targetName = prizeName === '谢谢参与' ? prizeName : prizeStock ? `恭喜您，获得 ${prizeName}` : '很抱歉，您来晚了，当前奖品已无库存'
      },
      // 抽奖转盘绘制完成
      handleDrawFinish () {
        console.log('抽奖转盘绘制完成')
        uni.hideLoading()
        uni.showToast({
          title: '奖品准备就绪'
        })
      }
    },
    onLoad () {
      // 模拟请求奖品数据
      this.getPrizeList()
    },
    onUnload () {
      uni.hideLoading()
    }
  }
</script>

<style lang="scss" scoped>
  $wheelBgUrl: '~static/almost-lottery/almost-lottery__bg';
  .home {
    padding: 50px 0;
  }
  .home-title {
    text-align: center;
    margin-bottom: 50px;
  }
  .home-title__text {
    color: #000;
    font-size: 30px;
    font-weight: bold;
  }
  .home-result {
    text-align: center;
    margin-top: 50px;
  }
  .home-result__text {
    color: #000;
    font-size: 20px;
    font-weight: bold;
  }
  .home-wheel {
    text-align: center;
  }
  .almost-lottery__tip {
    font-size: 24rpx;
  }
  .home-action {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 40rpx;
    width: 260rpx;
    height: 80rpx;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    border-radius: 10rpx;
    text-align: center;
    background-color: red;
    margin: 0 auto;
  }
  .home-action__text {
    color: #FFFFFF;
    font-size: 24rpx;
  }
</style>

<template>
  <div class="goods">
    <!-- 左侧导航栏 -->
    <div class="menu-wrapper" ref="menuWrapper" >
      <ul>
        <li v-for="(item,index) in goods" :key="index" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!-- 右侧商品栏 ref是供vue获取dom-->
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="(item,index) in goods" :key="index" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="(food,index) in item.foods" :key="index" class="food-item border-1px">
              <div class="icon">
                <img width="57" height="57" :src="food.icon" >
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <!-- 商品售卖情况 -->
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <!-- 价格 -->
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="cartAdd" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart ref='shopcart' :selectFoods="selectFoods" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"></shopcart>
    <!-- ref调用子组件 @add是子组件派发的事件，父组件监听这个事件 -->
    <food @add="cartAdd" :food="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  // 滚动栏插件
  import BScroll from 'better-scroll';
  // 引入购物车组件
  import shopcart from '@/components/shopcart/shopcart';
  // 引入添加至购物车的按钮
  import cartcontrol from '@/components/cartcontrol/cartcontrol';
  // 引入商品详情页
  import food from '@/components/food/food';
  const ERR_OK = 0;
  export default {
    // 绑定数据
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    computed: {
      // 实现页面的滚动效果
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      // 通过两层循环可以得出所有被选中的foods，然后传给shopcart可实现两个组件的联动
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
            good.foods.forEach((food) => {
              if (food.count) {
                foods.push(food);
              }
            });
        });
        return foods;
      }
    },
    created() {
      this.classMap = ["decrease", "discount", "special", "invoice", "guarantee"];
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          // 获取数据是异步的，一开始没有数据的dom高度是错误的，所以需要保证原生dom渲染完毕
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      // 实现左右联动效果
      selectMenu(index, event) {
        // 监听到原生点击事件的时候直接return
        if (!event._constructed) {
          return;
        };
         let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
         let el = foodList[index];
         this.foodsScroll.scrollToElement(el, 300);
      },
      selectFood(food,event) {
         if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      },
      _initScroll() {
        // 两个参数，一个dom对象，一个是options
        this.menuScroll = new BScroll(this.$refs.menuWrapper,{
          // 点击事件必须传入click：true，否则点击无效果（默认派发一个click事件）
          click: true
        });
        // 监听实时滚动的位置
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{
          click: true,
          probeType: 3
        });
        this.foodsScroll.on('scroll',(pos) => {
          // 取到的滚动值是负值，需要手动转换成正值
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      // 计算区间高度完成菜单和商品的滚动联动
      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        // 表示每一个区间的高度
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },
      _drop(target) {
        this.$nextTick(() => {
          this.$refs['shopcart'].drop(target);
        });
      },
      // 动画小球，events被弃用，所以写在methods里面
      cartAdd(target) {
        this._drop(target);
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "~@/common/stylus/mixin.styl";

  .goods
    display flex
    position absolute
    top 174px
    bottom 46px
    width 100%
    overflow hidden
    .menu-wrapper
      flex 0 0 80px
      width 80px
      background #f3f5f7
      .menu-item
        display table
        height 54px
        width 56px
        padding 0 12px
        line-height 14px
        &.current
          position relative
          z-index 10
          margin-top -1px
          background #ffffff
          font-weight 700
          .text
            border-none()
        .icon
          display inline-block;
          width 12px
          height 12px
          margin-right 4px
          background-size 12px 12px
          background-repeat no-repeat
          // 根据不同样式展示不同icon
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display table-cell
          width 56px
          vertical-align middle
          border-1px(rgba(7,17,27,0.1))
          font-size 12px
    .foods-wrapper
      flex 1
      .title
        padding-left 14px
        height 26px
        line-height 26px
        border-left 2px solid #d9dde1
        font-size 12px
        color rgb(147,153,159)
        background #f3f5f7
      .food-item
        display flex
        margin 18px
        padding-bottom 18px
        border-1px(rgba(7,17,27,0.1))
        &:last-child
          border-none()
          margin-bottom 0
        .icon
          flex 0 0 57px
          margin-right 10px
        .content
          flex 1
          .name
            margin 2px 0 8px 0
            height 14px
            line-height 14px
            font-size 14px
            color rgb(7,17,27)
          .desc, .extra
            line-height 10px
            font-size 10px
            color rgb(147,153,159)
          .desc
            line-height 12px
            margin-bottom 8px
          .extra
            .count
              margin-right 12px
          .price
            font-weight 700
            line-height 24px
            .now
              margin-right 8px
              font-size 14px
              color rgb(240,20,20)
            .old
              text-decoration line-through
              font-size 10px
              color rgb(147,153,159)
          .cartcontrol-wrapper
            position absolute
            right 0
            bottom 12px
</style>

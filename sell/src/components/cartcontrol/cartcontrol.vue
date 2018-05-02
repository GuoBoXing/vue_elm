<template>
  <div class="cartcontrol">
    <transition name="move">
      <div class="cart-dectease" @click.stop.prevent="decteaseCart" v-show="food.count>0">
        <span class="inner icon-remove_circle_outline"></span>
      </div>
    </transition>
    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>

<script type="text/ecmascript-6">
// 如果没有某个字段，直接赋值是不可以的，需要用到vue的组件
import Vue from 'vue';
export default {
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    addCart(event) {
      if (!event._constructed) {
        return;
      }
      if (!this.food.count) {
        Vue.set(this.food,'count',1);
      } else {
        this.food.count++;
      }
      // 动画小球
      this.$emit('add',event.target);
    },
    decteaseCart(event) {
      if (!event._constructed) {
        return;
      }
      if (this.food.count) {
        this.food.count--;
      }
    }
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .cartcontrol
    font-size 0
    .cart-dectease
      display inline-block
      padding 6px
      transition all 0.4s linear
      &.move-transition
        opacity 1
        transform translate3d(0,0,0)
      .inner
        display inline-block
        line-height 24px
        font-size 24px
        color rgb(0,160,220)
        transition all 0.4s linear
        transform rotate(0)
        // 动画进入和动画移出的时候
      &.move-enter, &.move-leave
        opacity 0
        // 平移动画
        transform translate3d(24px,0,0)
        // 滚动动画
        .inner
           transform rotate(180deg)
    .cart-count
      display inline-block
      vertical-align top
      width 12px
      padding-top 6px
      line-height 24px
      text-align center
      font-size 10px
      color rgb(147,153,159)
    .cart-add
      display inline-block
      color rgb(0,160,220)
      padding 6px
      line-height 24px
      font-size 24px
</style>

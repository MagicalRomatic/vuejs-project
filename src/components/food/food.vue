<template>
  <transition name="move">
    <div v-show="showFlag" class="food" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol @add="addFood" :food="food"></cartcontrol>
          </div>
          <transition name="fade">
            <div @click.stop.prevent="addFirst" class="buy" v-show="!food.count || food.count===0">
              加入购物车
            </div>
          </transition>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect @select="selectRating" @toggle="toggleContent" :selectType="selectType"
                        :onlyContent="onlyContent" :desc="desc"
                        :ratings="food.ratings"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-show="needShow(rating.rateType,rating.text)" v-for="rating in food.ratings"
                  class="rating-item border-1px">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img class="avatar" width="12" height="12" :src="rating.avatar">
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import Vue from 'vue';
  import {formatDate} from 'common/js/date';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import ratingselect from 'components/ratingselect/ratingselect';
  import split from 'components/split/split';

  const ALL = 2;

  export default {
    props: {
      food: {
        type: Object
      }
    },
    data() {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      };
    },
    methods: {
      show() {
        this.showFlag = true;
        this.selectType = ALL;
        this.onlyContent = true;
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.food, {
              click: true
            });
          } else {
            this.scroll.refresh();
          }
        });
      },
      hide() {
        this.showFlag = false;
      },
      addFirst(event) {
        if (!event._constructed) {
          return;
        }
        this.$emit('add', event.target);
        Vue.set(this.food, 'count', 1);
      },
      needShow(type, text) {
        if (this.onlyContent && !text) {
          return false;
        }
        if (this.selectType === ALL) {
          return true;
        } else {
          return type === this.selectType;
        }
      },
      addFood(target) {
        this.$emit('add', target);
      },
      selectRating(type) {
        this.selectType = type;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      toggleContent() {
        this.onlyContent = !this.onlyContent;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    components: {
      cartcontrol,
      ratingselect,
      split
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .food
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    z-index: 30
    width: 100%
    background: #f8f9fa // 轻柔的背景色
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1) // 阴影效果
    border-radius: 8px // 圆角效果
    transform: translate3d(0, 0, 0)
    
    &.move-enter-active, &.move-leave-active
      transition: all 0.2s ease-in-out // 动画过渡更平滑
    &.move-enter, &.move-leave-active
      transform: translate3d(100%, 0, 0)
      
    .image-header
      position: relative
      width: 100%
      height: 0
      padding-top: 100%
      border-top-left-radius: 8px // 圆角
      border-top-right-radius: 8px // 圆角
      img
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
        border-top-left-radius: 8px // 圆角
        border-top-right-radius: 8px // 圆角
      
      .back
        position: absolute
        top: 10px
        left: 10px
        .icon-arrow_lift
          display: block
          padding: 10px
          font-size: 24px // 字体增大
          color: rgba(255, 255, 255, 0.9) // 透明度更高

    .content
      position: relative
      padding: 20px // 增加内边距
      .title
        line-height: 20px // 增加行高
        margin-bottom: 10px
        font-size: 16px // 字体增大
        font-weight: bold
        color: #343a40 // 更深的颜色，提升可读性
      
      .detail
        margin-bottom: 18px
        line-height: 14px
        height: auto // 允许自适应高度
        .sell-count, .rating
          font-size: 12px // 字体增大
          color: #6c757d // 统一的字体颜色
        .sell-count
          margin-right: 12px
      
      .price
        font-weight: bold
        line-height: 28px // 增加行高
        .now
          margin-right: 8px
          font-size: 18px // 字体增大
          color: #e63946 // 突出的价格颜色
        .old
          text-decoration: line-through
          font-size: 14px // 字体增大
          color: #adb5bd // 更柔和的颜色
      
      .cartcontrol-wrapper
        position: absolute
        right: 12px
        bottom: 12px
      
      .buy
        position: absolute
        right: 18px
        bottom: 18px
        z-index: 10
        height: 36px // 增加按钮高度
        line-height: 36px // 增加行高
        padding: 0 16px // 增加内边距
        box-sizing: border-box
        border-radius: 18px // 圆角按钮
        font-size: 12px // 字体增大
        color: #fff
        background: #007bff // 更吸引人的蓝色
        opacity: 1
        
        &.fade-enter-active, &.fade-leave-active
          transition: all 0.2s
        &.fade-enter, &.fade-leave-active
          opacity: 0
          z-index: -1
    
    .info
      padding: 20px // 增加内边距
      .title
        line-height: 16px
        margin-bottom: 8px
        font-size: 16px // 字体增大
        color: #343a40 // 更深的颜色
      
      .text
        line-height: 24px
        padding: 0 8px
        font-size: 14px // 字体增大
        color: #495057 // 更柔和的颜色
    
    .rating
      padding-top: 18px
      .title
        line-height: 16px
        margin-left: 18px
        font-size: 16px // 字体增大
        color: #343a40 // 更深的颜色
      
      .rating-wrapper
        padding: 0 18px
        .rating-item
          position: relative
          padding: 16px 0
          border-bottom: 1px solid rgba(7, 17, 27, 0.1) // 使用底部边框
          .user
            position: absolute
            right: 0
            top: 16px
            line-height: 12px
            font-size: 12px // 字体增大
            color: #6c757d // 更柔和的颜色
            .name
              display: inline-block
              margin-right: 6px
              vertical-align: top
              font-size: 12px // 字体增大
              color: #6c757d
            .avatar
              border-radius: 50%
              border: 1px solid #e9ecef // 头像边框
      
          .time
            margin-bottom: 6px
            line-height: 12px
            font-size: 12px // 字体增大
            color: #6c757d // 更柔和的颜色
          
          .text
            line-height: 16px
            font-size: 14px // 字体增大
            color: #343a40 // 更深的颜色
            .icon-thumb_up, .icon-thumb_down
              margin-right: 4px
              line-height: 16px
              font-size: 14px // 字体增大
            .icon-thumb_up
              color: #007bff // 使用主题颜色
            .icon-thumb_down
              color: #6c757d // 更柔和的颜色

        .no-rating
          padding: 16px 0
          font-size: 12px
          color: #6c757d // 更柔和的颜色

</style>
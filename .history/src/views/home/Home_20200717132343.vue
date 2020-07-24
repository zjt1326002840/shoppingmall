<template>
  <!-- 首页 -->
  <div id='home'>
    <!-- 导航栏 -->
    <nav-bar class="home-nav">
      <div slot="center">购 物 街</div>
    </nav-bar>
    <tab-control ref="tabControl1" :titles="['流行', '新款', '精选']" @tabClick="tabClick" class="tab-control-copy"
      v-show="isTabFixed">
    </tab-control>
    <scroll class="scroll" ref="scroll" :probe-type="3" @scroll="contentScroll" :pull-up-load="true"
      @pullingUp="loadMore">
      <!-- 轮播图 -->
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <!-- 推荐信息 -->
      <recommend-view :recommends="recommends"></recommend-view>
      <!-- 本周流行 -->
      <feature-view></feature-view>
      <!-- 选项卡 -->
      <tab-control ref="tabControl" :titles="['流行', '新款', '精选']" @tabClick="tabClick">
      </tab-control>
      <!-- 推荐展示 -->
      <goods-list :goods="showGoods">
        <!-- 展示子选项卡 -->
      </goods-list>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
  // 导入公共组件
  import NavBar from "components/common/navbar/NavBar"; // 导航栏
  import TabControl from "components/content/tabControl/TabControl"; // 选项卡
  import GoodsList from "components/content/goods/GoodsList"; // 推荐展示
  import Scroll from "components/common/scroll/Scroll"; // 滚动
  import BackTop from "components/content/backTop/BackTop"; // 返回顶部

  // 导入子组件
  import HomeSwiper from "./childComps/HomeSwiper"; // 轮播图
  import RecommendView from "./childComps/RecommendView"; // 推荐信息
  import FeatureView from "./childComps/FeatureView"; // 本周流行

  // 导入方法, 数据
  import { getHomeMultidata, getHomeGoods } from "network/home";
  import { debounce } from "common/utils"; // 防抖函数

  export default {
    name: "Home",
    components: {
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop,
      HomeSwiper,
      RecommendView,
      FeatureView
    },
    data() {
      return {
        banners: [],
        recommends: [],
        goods: {
          pop: {
            page: 0,
            list: []
          },
          new: {
            page: 0,
            list: []
          },
          sell: {
            page: 0,
            list: []
          }
        },
        currentType: "pop",
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
        saveY: 0
      };
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list;
      }
    },
    activated() {
      this.$refs.scroll.refresh();
      this.$refs.scroll.scrollTo(0, this.saveY, 0);
    },
    deactivated() {
      this.saveY = this.$refs.scroll.scroll.y;
    },
    created() {
      // 1.请求多个数据
      this.getHomeMultidata();

      // 2.请求商品数据
      this.getHomeGoods("pop");
      this.getHomeGoods("new");
      this.getHomeGoods("sell");
    },
    mounted() {
      // 监听item中图片加载完成
      const refresh = debounce(this.$refs.scroll.refresh, 100);
      this.$bus.$on("itemImageLoad", () => {
        refresh();
      });
    },
    methods: {
      /**
       * 事件监听相关的方法
       */
      tabClick(index) {
        switch (index) {
          case 0:
            this.currentType = "pop";
            break;
          case 1:
            this.currentType = "new";
            break;
          case 2:
            this.currentType = "sell";
            break;
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl.currentIndex = index;
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0, 500);
      },
      contentScroll(position) {
        // 1.判断BackTop是否显示
        this.isShowBackTop = -position.y > 1000;

        // 2.决定tabControl是否吸顶 (position: fixed)
        this.isTabFixed = -position.y >= this.tabOffsetTop;
      },
      loadMore() {
        this.getHomeGoods(this.currentType);
        this.$refs.scroll.finishPullUp();
      },
      swiperImageLoad() {
        // 获取tabControl的offsetTop
        // 所有的组件都有一个属性 $el: 用于获取组件中元素
        this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop;
      },

      /**
       * 网络请求相关的方法
       */
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          // console.log(res);
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
        });
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1;
        getHomeGoods(type, page).then(res => {
          // console.log(res);
          this.goods[type].list.push(...res.data.list);
          this.goods[type].page++;
        });
      }
    }
  };
</script>

<style scoped>
  #home {
    position: relative;
    height: 100vh;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

    /* position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9;
  font-size: 18px; */
  }

  .scroll {
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

  .tab-control-copy {
    position: relative;
    z-index: 9;
  }
</style>
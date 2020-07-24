<template>
  <!-- 轮播图组件 -->
  <swiper>
    <swiper-item v-for="(item, index) in banners"
                 :key="index"
                 class="swiper_item">
      <a :href="item.link">
        <img :src="item.image"
             alt=""
             @load="imageLoad">
      </a>
    </swiper-item>
  </swiper>
</template>

<script>
import { Swiper, SwiperItem } from "components/common/swiper";

export default {
  name: "HomeSwiper",
  props: {
    banners: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  data() {
    return {
      isLoad: true
    };
  },
  components: {
    Swiper,
    SwiperItem
  },
  methods: {
    imageLoad() {
      if (this.isLoad) {
        this.$emit("swiperImageLoad");
        this.isLoad = false;
      }
    }
  }
};
</script>

<style scoped>
</style>
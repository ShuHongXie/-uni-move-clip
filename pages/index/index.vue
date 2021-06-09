<template>
  <view class="content">
    <image :src="url" v-if="url" mode="widthFix"></image>
    <l-clipper
      v-if="show"
      @success="
        url = $event.url;
        show = false;
      "
      @cancel="show = false"
    />
    <button @tap="show = true">裁剪</button>
  </view>
</template>

<script>
// uni_modules引入
import lClipper from "@/uni_modules/lime-clipper/components/lime-clipper/";
export default {
  components: { lClipper },
  data() {
    return {
      title: "Hello",
      show: false,
      url: "https://t7.baidu.com/it/u=1595072465,3644073269&fm=193&f=GIF",
    };
  },
  onLoad() {},
  methods: {},
};
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin-top: 200rpx;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 50rpx;
}

.text-area {
  display: flex;
  justify-content: center;
}

.title {
  font-size: 36rpx;
  color: #8f8f94;
}
</style>

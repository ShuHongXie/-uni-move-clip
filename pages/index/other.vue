<!--
 * @Author: shuhongxie
 * @Date: 2021-05-28 11:52:18
 * @LastEditors: 谢树宏
 * @LastEditTime: 2021-06-01 10:54:34
 * @FilePath: /uni-test/pages/index/other.vue
-->
<template>
  <div class="clip">
    <img :src="imageUrl" alt="" class="example--image" />
    <div
      class="clip__content"
      :style="style"
      @touchstart.stop.prevent="clipTouchStart"
      @touchmove.stop.prevent="clipTouchMove"
    >
      <div
        v-for="(item, index) in 4"
        :key="index"
        class="clip__edge"
        @touchstart.stop.prevent="edgeTouchStart"
        @touchmove.stop.prevent="(e) => edgeTouchMove(e, index)"
        @touchend.stop.prevent="edgeTouchEnd"
      ></div>
      <canvas class="clip-canvas" canvas-id="clip-canvas"></canvas>
    </div>
    <button class="test-btn" @click="initCanvas">测试图片</button>
    <img class="test-image" :src="url" alt="" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      clipStyle: {},
      clientX: 0,
      clientY: 0,
      touchX: 0,
      touchY: 0,
      left: 0,
      top: 0,
      width: 0,
      height: 0,
      url: "",
      canvasInstance: null,
      imageUrl: "https://t7.baidu.com/it/u=1595072465,3644073269&fm=193&f=GIF",
      systemInfo: null,
      screenHeight: 0,
    };
  },
  computed: {
    style() {
      let style = "";
      for (let key in this.clipStyle) {
        style += `${key}:${this.clipStyle[key]}px;`;
      }
      return style;
    },
  },
  async mounted() {
    this.setStyle();
    uni.getSystemInfo({
      success: (res) => {
        console.log(res);
        this.systemInfo = res;
      },
    });
    uni
      .createSelectorQuery()
      .select(".clip__content")
      .fields(
        {
          size: true,
        },
        (data) => {
          this.width = data.width;
          this.height = data.height;
        }
      )
      .exec();
    uni
      .createSelectorQuery()
      .select(".clip")
      .fields(
        {
          size: true,
        },
        (data) => {
          this.screenHeight = data.height;
        }
      )
      .exec();
  },
  methods: {
    clipTouchStart(e) {
      console.log("点击了本体");
      this.touchX = e.changedTouches[0].pageX;
      this.touchY = e.changedTouches[0].pageY;
    },
    clipTouchMove(e) {
      console.log("本体移动", e);
      const { screenWidth } = this.systemInfo;
      const currX = e.changedTouches[0].pageX;
      const currY = e.changedTouches[0].pageY;
      const moveX = currX - this.touchX;
      const moveY = currY - this.touchY;
      this.touchX = currX;
      this.touchY = currY;
      if (this.left + moveX < 0) {
        this.left = 0;
      } else if (this.left + moveX > screenWidth - this.width) {
        this.left = screenWidth - this.width;
      } else {
        this.left = this.left + moveX;
      }
      console.log(this.top, moveY);
      if (this.top + moveY < 0) {
        this.top = 0;
      } else if (this.top + moveY > this.screenHeight - this.height) {
        this.top = this.screenHeight - this.height;
      } else {
        this.top = this.top + moveY;
      }
      this.clipStyle = {
        ...this.clipStyle,
        left: this.left,
        top: this.top,
      };
    },
    edgeTouchStart(e) {
      console.log("点击了四方格");
      this.clientX = e.changedTouches[0].clientX;
      this.clientY = e.changedTouches[0].clientY;
    },
    edgeTouchMove(e, index) {
      const currX = e.changedTouches[0].clientX;
      const currY = e.changedTouches[0].clientY;
      const moveX = currX - this.clientX;
      const moveY = currY - this.clientY;
      this.clientX = currX;
      this.clientY = currY;
      const { width, height, left, top, screenHeight } = this;
      const { screenWidth } = this.systemInfo;
      let maxWidth = 0,
        maxHeight = 0,
        maxTop = top + moveY < 0 ? 0 : top + moveY,
        maxLeft = left + moveX < 0 ? 0 : left + moveX;
      // if (!maxLeft || !maxTop) return;
      if (index % 2 === 0) {
        maxWidth = width - moveX > screenWidth ? screenWidth : width - moveX;
      } else {
        maxWidth = width + moveX > screenWidth ? screenWidth : width + moveX;
      }
      if (index < 2) {
        maxHeight =
          height - moveY > screenHeight ? screenHeight : height - moveY;
      } else {
        maxHeight =
          height + moveY > screenHeight ? screenHeight : height + moveY;
      }

      if (index === 0) {
        if (width - moveX <= 40 || height - moveY <= 40) return;
        console.log(maxLeft);
        this.clipStyle = {
          width: maxWidth,
          height: maxHeight,
          left: maxLeft,
          top: maxTop,
        };
        this.width = maxWidth;
        this.height = maxHeight;
        this.top = maxTop;
        this.left = maxLeft;
        // 右上角
      } else if (index === 1) {
        if (width + moveX <= 40 || height - moveY <= 40) return;
        this.clipStyle = {
          width: maxWidth,
          height: maxHeight,
          left,
          top: maxTop,
        };

        this.width = maxWidth;
        this.height = maxHeight;
        this.top = maxTop;
      } else if (index === 2) {
        if (width - moveX <= 40 || height + moveY <= 40) return;
        this.clipStyle = {
          width: maxWidth,
          height: maxHeight,
          left: maxLeft,
          top,
        };

        this.width = maxWidth;
        this.height = maxHeight;
        this.left = maxLeft;
      } else if (index === 3) {
        if (width + moveX <= 40 || height + moveY <= 40) return;
        this.clipStyle = {
          width: maxWidth,
          height: maxHeight,
          left,
          top,
        };

        this.width = maxWidth;
        this.height = maxHeight;
      }
      console.log(this.left, this.top);
    },
    edgeTouchEnd() {},
    initCanvas() {
      uni.showLoading({
        title: "加载中...",
      });
      uni
        .createSelectorQuery()
        .select(".clip__content")
        .fields(
          {
            size: true,
            scrollOffset: true,
            rect: true,
            context: true,
            computedStyle: ["transform", "translateX"],
            scrollOffset: true,
          },
          (data) => {
            uni.downloadFile({
              url: this.imageUrl,
              success: (res) => {
                this.canvasInstance = uni.createCanvasContext(
                  "clip-canvas",
                  this
                );
                this.canvasInstance.drawImage(
                  res.tempFilePath,
                  -data.left,
                  -data.top,
                  this.systemInfo.screenWidth,
                  this.screenHeight,
                  0,
                  0
                );
                this.canvasInstance.draw(
                  false,
                  (() => {
                    setTimeout(() => {
                      uni.canvasToTempFilePath(
                        {
                          x: 0,
                          y: 0,
                          width: data.width,
                          height: data.height,
                          dWidth: data.width,
                          dHeight: data.height,
                          fileType: "jpg",
                          canvasId: "clip-canvas",
                          success: (data) => {
                            uni.hideLoading();

                            this.url = data.tempFilePath;
                            // this.canvasInstance.save();
                          },
                        },
                        this
                      );
                    }, 500);
                  })()
                );
              },
            });
          }
        )
        .exec();
    },
    setStyle() {
      uni
        .createSelectorQuery()
        .select(".clip__content")
        .fields(
          {
            size: true,
            scrollOffset: true,
          },
          (data) => {
            this.width = data.width;
            this.height = data.height;
          }
        )
        .exec();
    },
  },
};
</script>

<style lang="scss" scoped>
$edge-border-width: 0rpx;
.example--image {
  width: 100vw;
  height: 100vh;
}
.test-btn {
  position: fixed;
  left: 0;
  right: 0;
  margin: 0 auto;
  bottom: 300rpx;
  z-index: 999;
}
.test-image {
  position: fixed;
  left: 0;
  right: 0;
  margin: 0 auto;
  bottom: 200rpx;
  // width: 100%;
  // height: 300rpx;
  z-index: 998;
}
.clip {
  .move {
    &-area {
      width: 100vw;
      height: 100vh;
      position: relative;
      overflow: hidden;
    }
    &-view {
      position: absolute;
      width: 400rpx;
      height: 400rpx;
    }
  }
  &-canvas {
    // position: fixed;
    width: 100%;
    height: 100%;
    transform: translate(-200vw, -200vw);
    // left: -200vw;
    // top: -200vw;
    // pointer-events: none;
  }
  &__content {
    position: fixed;
    // left 6rpx
    width: 400rpx;
    height: 400rpx;
    left: 0;
    top: 0;
    // pointer-events: auto;
    border: 1px solid red;
    z-index: 4;
    overflow: hidden;
    box-shadow: rgba(0, 0, 0, 0.5) 0 0 0 200vh;
  }
  &__edge {
    position: absolute;
    // left 6rpx
    width: 34rpx;
    height: 34rpx;
    border: 10rpx solid red;
    pointer-events: auto;
    z-index: 2;
    &::before {
      content: "";
      position: absolute;
      z-index: 2;
      width: 40rpx;
      height: 40rpx;
      background-color: transparent;
    }
    &:nth-child(1) {
      left: $edge-border-width;
      top: $edge-border-width;
      border-bottom-width: 0 !important;
      border-right-width: 0 !important;
      &:before {
        top: -50%;
        left: -50%;
      }
    }
    &:nth-child(2) {
      right: $edge-border-width;
      top: $edge-border-width;
      border-bottom-width: 0 !important;
      border-left-width: 0 !important;
      &:before {
        top: -50%;
        left: 50%;
      }
    }
    &:nth-child(3) {
      left: $edge-border-width;
      bottom: $edge-border-width;
      border-top-width: 0 !important;
      border-right-width: 0 !important;
      &:before {
        bottom: -50%;
        left: -50%;
      }
    }
    &:nth-child(4) {
      right: $edge-border-width;
      bottom: $edge-border-width;
      border-top-width: 0 !important;
      border-left-width: 0 !important;
      &:before {
        bottom: -50%;
        left: 50%;
      }
    }
  }
}
</style>

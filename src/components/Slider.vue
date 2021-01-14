<template>
  <div
    @click="handleClick"
    :class="['mx-slider-wrap', disabled ? 'disabled' : '']"
  >
    <div class="mx-dots" v-if="showdots && dotsInfo.length > 0">
      <span
        v-for="(v, index) in dotsInfo"
        :class="[
          'mx-dots-item',
          v.left > processPercent ? '' : 'mx-dots-choose',
        ]"
        :key="index"
      >
        {{ v.text }}
      </span>
    </div>
    <div
      ref="slider"
      class="mx-slider"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
      @touchcancel="handleTouchEnd"
    >
      <div
        class="mx-slider__bar"
        :style="{
          width: `${processPercent}%`,
        }"
      >
        <div class="mx-slider__bar-wrapper">
          <div class="mx-slider__bar-button"></div>
        </div>
      </div>
      <div class="mx-slider__steps" v-if="showdots && dotsInfo.length > 0">
        <span
          v-for="(v, index) in dotsInfo"
          :class="[
            'mx-slider__steps-item',
            v.left > processPercent ? '' : 'mx-slider__steps-choose',
          ]"
          :key="index"
          :style="{ left: `${v.left}%` }"
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MxSlider",
  props: {
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    value: {
      type: Number,
      default: 0,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    step: {
      type: Number,
      default: 1,
    },
    isStaticStep: {
      type: Boolean,
      default: false,
    },
    dots: {
      type: Array,
      default: () => [],
    },
    showdots: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      startX: 0,
      startY: 0,
      currentX: 0,
      currentY: 0,
      currentMin: this.min,
      currentMax: this.max,
      sliderWidth: 0,
      sliderLeft: 0,
      currentValue: this.value,
      draging: false,
      clicking: false,
      processPercent: 0,
    };
  },
  computed: {
    range() {
      return this.currentMax - this.currentMin;
    },
    dotsInfo() {
      const len = this.dots.length;
      if (this.isStaticStep) {
        return this.dots.map((item, index) => {
          return {
            ...item,
            left: (100 / (len - 1)) * index,
          };
        });
      } else {
        return [
          {
            text: (this.dots[0] && this.dots[0].text) || this.min,
            left: 0,
          },
          {
            text: (this.dots[len - 1] && this.dots[len - 1].text) || this.max,
            left: 100,
          },
        ];
      }
    },
  },
  watch: {
    value: {
      handler(val) {
        this.setProcessPercent(val);
      },
      deep: true,
    },
    currentValue: {
      handler(nVal, oVal) {
        if (nVal !== oVal && this.clicking) {
          this.$emit("change", this.currentValue);
          this.clicking = false;
        }
      },
    },
  },
  mounted() {
    const rect = this.$refs.slider.getBoundingClientRect();
    this.sliderWidth = rect.width;
    this.sliderOffsetX = rect.left;
    this.setProcessPercent(this.value);
  },
  methods: {
    handleClick(e) {
      e.stopPropagation();
      if (this.disabled) {
        return false;
      }
      this.currentValue = this.updateValue(e.pageX);
      this.setProcessPercent(this.currentValue);
      this.clicking = true;
      // if (this.currentValue !== this.value) {
      //   this.$emit("change", this.currentValue);
      // }
    },
    handleTouchStart(e) {
      if (this.disabled) {
        return false;
      }
      // console.log("start", e);
      this.draging = false;
      const touches = e.changedTouches[0];
      this.startX = touches.pageX;
      this.startY = touches.pageY;
    },
    handleTouchMove(e) {
      // console.log("move", e);
      e.preventDefault();
      if (this.disabled) {
        return false;
      }
      this.draging = true;
      const touches = e.changedTouches[0];
      this.currentX = touches.pageX;
      this.currentValue = this.updateValue(this.currentX);
      this.setProcessPercent(this.currentValue);
      this.$emit("input", this.currentValue);
    },
    handleTouchEnd() {
      if (this.disabled) {
        return false;
      }
      if (this.draging) {
        this.$emit("change", this.currentValue);
      }
      this.draging = false;
    },
    updateValue(val) {
      if (this.isStaticStep) {
        return this.getJSONObj(val)["value"];
      } else {
        const currentVal =
          ((val - this.sliderOffsetX) / this.sliderWidth) * this.range +
          this.min;
        return this.format(currentVal);
      }
    },
    format(value) {
      return (
        Math.round(Math.max(this.min, Math.min(value, this.max)) / this.step) *
        this.step
      );
    },
    setProcessPercent(curVal) {
      if (this.isStaticStep) {
        this.processPercent = this.dotsInfo.find(
          (item) => item.value === curVal
        ).left;
      } else {
        this.processPercent = ((curVal - this.min) * 100) / this.range;
      }
    },
    getJSONObj(val) {
      const currentVal = ((val - this.sliderOffsetX) / this.sliderWidth) * 100;
      const len = this.dots.length;
      const oneStepPercent = 100 / (len - 1);
      // 1是最小值
      const currentStep = Math.round(currentVal / oneStepPercent) + 1;
      return this.dotsInfo[currentStep - 1];
    },
  },
};
</script>

<style lang="less">
.mx-slider-wrap {
  padding: 16px;
}
.mx-slider {
  width: 100%;
  margin: 12px 0;
  height: 2px;
  background: rgba(255, 255, 255, 0.2);
  position: relative;
  user-select: none;
  &:before {
    position: absolute;
    top: -8px;
    right: 0;
    bottom: -8px;
    left: 0;
    content: "";
  }
  &__bar {
    background: #fff;
    width: 0%;
    height: 100%;
    position: relative;
    z-index: 1;
    &-wrapper {
      position: absolute;
      right: 0;
      top: 50%;
      transform: translateY(-50%);
    }
    &-button {
      width: 24px;
      height: 24px;
      border-radius: 50%;
      box-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
      transform: translateX(50%);
      background: #ffffff;
    }
  }
  &__steps {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 2px;
    &-item {
      height: 3px;
      width: 3px;
      border-radius: 50%;
      background: #fff;
      display: inline-block;
      position: absolute;
      top: 50%;
      left: 0;
      transform: translate(-50%, -50%);
    }
    &-choose {
      background: #ccc;
    }
  }
}
.mx-dots {
  // position: absolute;
  // left: 0;
  // top: 26px;
  padding: 8px 0;
  width: 100%;
  height: 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  &-item {
    color: rgba(255, 255, 255, 0.6);
    font-size: 14px;
    // transform: translateY(25px);
    // transform: translate(-50%, -50%);
    &:first-child {
      transform: translateX(-50%);
    }
    &:last-child {
      transform: translateX(50%);
    }
  }
}
.disabled {
  .mx-slider {
    opacity: 0.5;
  }
}
</style>

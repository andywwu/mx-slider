<template>
  <div id="app">
    <div class="range">
      <Slider
        @change="handleChange"
        :min="10"
        :value="1"
        :dots="list"
        :showdots="true"
        :isStaticStep="false"
        :disabled="true"
      />
    </div>
  </div>
</template>

<script>
import Slider from "./components/Slider.vue";

export default {
  name: "App",
  components: {
    Slider,
  },
  data() {
    return {
      list: [
        {
          text: "一档",
          value: 1,
        },
        {
          text: "二档",
          value: 2,
        },
        {
          text: "三档",
          value: 3,
        },
      ],
    };
  },
  methods: {
    handleChange(e) {
      console.log("滑动结束", e);
    },
    // handleInput(e) {
    //   console.log("滑动中", e);
    // },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
  background: linear-gradient(180deg, #183b56 0%, #0475cc 100%);
}
.range {
  padding: 20px;
}
</style>

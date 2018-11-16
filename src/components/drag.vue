<template>
  <div :class="$style.container">
    <div
      :class="`${$style.drag} ${cell}-${index}`"
      v-for="(item, index) in styleList"
      :key="index"
      :ref="`${cell}-${index}`"
      :style="item"
      :data-index="index"
      @mousedown.prevent.stop="start"
      @touchstart.prevent.stop="start"
      @mousemove.prevent.stop="move"
      @touchmove.prevent.stop="move"
      @mouseup.prevent.stop="end"
      @touchend.prevent.stop="end"
    >
      {{ htmlList[index] }}
    </div>
  </div>
</template>

<script>
export default {
  name: "VueDrag",
  props: {
    styleList: {
      type: Array,
      default: () => [
        {
          left: "50px",
          top: "50px",
          width: "50px",
          height: "50px",
          backgroundColor: "aqua",
          zIndex: 0
        }
      ]
    },
    htmlList: {
      type: Array,
      default: () => ["drag"]
    },
    moveZIndex: {
      type: Number,
      default: 100
    },
    cell: {
      type: String,
      default: "cell"
    },
    mutiTouch: {
      type: Boolean,
      default: false
    },
    changingDom: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      currMove: "",
      position: []
    };
  },
  methods: {
    start(e) {
      if (e.touches.length > 1 && !this.mutiTouch) return false;
      if (this.changingDom.includes(e.target)) return false;
      let dataIndex = e.target.dataset.index;
      let touch = [...e.touches].filter(
        i => i.target.dataset.index == dataIndex
      )[0];
      this.currMove = e.target;
      this.position[dataIndex] = {
        top: e.target.style.top,
        left: e.target.style.left,
        x: touch.pageX,
        y: touch.pageY
      };
      e.target.style.zIndex = this.moveZIndex;
      this.$emit("start", e);
    },
    move(e) {
      if (
        (e.touches.length > 1 || this.currMove !== e.target) &&
        !this.mutiTouch
      )
        return false;
      if (this.changingDom.includes(e.target)) return false;
      let dataIndex = e.target.dataset.index;
      let touch = [...e.touches].filter(
        i => i.target.dataset.index == dataIndex
      )[0];

      let moveX = touch.pageX - this.position[dataIndex].x;
      let moveY = touch.pageY - this.position[dataIndex].y;
      e.target.style.left =
        parseFloat(this.position[dataIndex].left) + moveX + "px";
      e.target.style.top =
        parseFloat(this.position[dataIndex].top) + moveY + "px";
      this.$emit("move", e, {
        top: e.target.style.top,
        left: e.target.style.left,
        x: touch.pageX,
        y: touch.pageY
      });
    },
    end(e) {
      if (e.target !== this.currMove && !this.mutiTouch) return false;
      if (this.changingDom.includes(e.target)) return false;
      let dataIndex = e.target.dataset.index;
      let touch = [...e.changedTouches].filter(
        i => i.target.dataset.index == dataIndex
      )[0];

      this.$emit("end", e, {
        top: e.target.style.top,
        left: e.target.style.left,
        x: touch.pageX,
        y: touch.pageY
      });
      // reset
      this.position[dataIndex] = null;
      this.currMove = "";
      e.target.style.zIndex = 0;
    }
  }
};
</script>

<style lang="less" module>
.container {
  .drag {
    position: absolute;
  }
}
</style>
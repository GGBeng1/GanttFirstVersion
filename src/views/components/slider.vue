<template>
  <div class="slider" ref="slider" :style="{ width: '100%' }">
    <div class="process" :style="{ width }"></div>
    <div class="thunk" ref="trunk" :style="{ left }">
      <div class="block">
        <i class="el-icon-caret-top" @mousedown.stop="thunkMousedown"></i>
      </div>
      <div class="tips" v-if="isMove">
        <span>{{ parseInt(scale * 100) }}</span>
        <i class="fas fa-caret-down"></i>
      </div>
    </div>
  </div>
</template>
<script>
/*
 * min 进度条最小值
 * max 进度条最大值
 * v-model 对当前值进行双向绑定实时显示拖拽进度
 * */
export default {
  props: ["min", "max", "value", "widths", "index"],
  data() {
    return {
      slider: null, //滚动条DOM元素
      thunk: null, //拖拽DOM元素
      per: this.value, //当前值
      isMove: false //当前是否拖拽tip
    };
  },
  methods: {
    thunkMousedown(e) {
      let _this = this;
      let width = parseInt(_this.width);
      let disX = e.clientX;
      this.$emit("thunkMousedown");
      this.isMove = true;
      document.onmousemove = function(e) {
        // value, left, width
        // 当value变化的时候，会通过计算属性修改left，width
        // 拖拽的时候获取的新width
        let newWidth = e.clientX - disX + width;
        // 拖拽的时候得到新的百分比
        let scale = newWidth / _this.slider.offsetWidth;
        _this.per = Math.ceil((_this.max - _this.min) * scale + _this.min);
        _this.per = Math.max(_this.per, _this.min);
        _this.per = Math.min(_this.per, _this.max);
        _this.$emit("thunkMousemove", e, this.index);
      };
      document.onmouseup = event => {
        this.isMove = false;
        this.$emit(
          "thunkMouseup",
          parseInt(this.scale * 100),
          event,
          this.index
        );
        document.onmousemove = document.onmouseup = null;
      };
      return false;
    }
  },
  //渲染到页面的时候
  mounted() {
    this.slider = this.$refs.slider;
    this.thunk = this.$refs.trunk;
  },
  computed: {
    // 设置一个百分比，提供计算slider进度宽度和trunk的left值
    // 对应公式为  当前值-最小值/最大值-最小值 = slider进度width / slider总width
    // trunk left =  slider进度width + trunk宽度/2
    scale() {
      return (this.per - this.min) / (this.max - this.min);
    },
    width() {
      if (this.slider) {
        return this.widths * this.scale + "px";
      } else {
        return 0 + "px";
      }
    },
    left() {
      if (this.slider) {
        return this.widths * this.scale - this.thunk.offsetWidth / 2 + "px";
      } else {
        return 0 + "px";
      }
    }
  }
};
</script>
<style>
.box {
  margin: 100px auto 0;
  width: 80%;
}
.clear:after {
  content: "";
  display: block;
  clear: both;
}
.slider {
  position: relative;
  height: 16px;
  background: #e4e7ed;
  border-radius: 3px;
  cursor: move;
  user-select: none;
}
.slider .process {
  position: absolute;
  left: 0;
  top: 0;
  width: 112px;
  height: 16px;
  border-radius: 3px;
  background: #409eff;
}
.slider .thunk {
  position: absolute;
  left: 100px;
  top: -7px;
  width: 20px;
  height: 20px;
}
.slider .block {
  transition: 0.2s all;
}
.slider .block i {
  font-size: 25px;
  position: relative;
  left: -3px;
  top: 15px;
  cursor: pointer;
}
.slider .tips {
  position: absolute;
  left: -7px;
  bottom: 30px;
  min-width: 15px;
  text-align: center;
  padding: 4px 8px;
  background: #000;
  border-radius: 5px;
  height: 24px;
  color: #fff;
}
.slider .tips i {
  position: absolute;
  margin-left: -5px;
  left: 50%;
  bottom: -9px;
  font-size: 16px;
  color: #000;
}
.slider .block:hover {
  transform: scale(1.1);
  opacity: 0.6;
}
</style>
---
layout: post
title: 放大镜
categories: Vue3
tags: [Vue3]
excerpt: Vue3放大镜组件
---

## 效果

![放大镜]({{ site.url }}{{site.baseurl}}/assets/images/post/20211214/magnifier.gif)

## 代码片段

### template

```vue
<template>
  <div class="magnifier-container">
    <div
      ref="magnificationElement"
      class="preview"
      :style="{ backgroundImage: 'url(' + options.src + ')' }"
    >
      <div ref="glass" class="magnifying-glass" :style="styleLarge"></div>
    </div>
  </div>
</template>
```

### script

```vue
<script>
export default {
  name: "Magnifier",
  props: {
    options: {
      type: Object,
      default: () => {
        return {
          src: String,
          srcLarge: String,
          width: {
            type: Number,
            default: 200,
          },
          height: {
            type: Number,
            default: 200,
          },
          shape: {
            type: String,
            default: "circular",
          },
          borderSize: {
            type: Number,
            default: 2,
          },
          borderColor: {
            type: String,
            default: "#666666",
          },
        };
      },
    },
  },
  computed: {
    styleLarge() {
      return {
        backgroundImage: "url(" + this.options.srcLarge + ")",
        backgroundPosition: this.backgroundPos,
        left: this.cursorX + "px",
        top: this.cursorY + "px",
        width: this.options.width + "px",
        height: this.options.height + "px",
        borderRadius: this.options.shape === "square" ? "0px" : "50%",
        transform:
          "translate(" +
          (-1 * this.options.width) / 2 +
          "px," +
          (-1 * this.options.width) / 2 +
          "px)",
        border: `${this.options.borderSize}px solid ${this.options.borderColor}`,
      };
    },
  },
  data: function () {
    return {
      bounds: null,
      cursorX: 0,
      cursorY: 0,
      thumbPos: { x: 0, y: 0 },
      backgroundPos: "0 0",
      largeWidth: null,
      largeHeight: null,
      ProportionX: null,
      ProportionY: null,
    };
  },
  mounted() {
    this.$nextTick(() => {
      this.$refs.magnificationElement.addEventListener(
        "mousemove",
        this.moveMagnifier
      );
    });
    this.getImgSize();
  },
  methods: {
    getImgSize() {
      const img = document.createElement("img");
      img.src = this.options.srcLarge;
      if (img.complete) {
        this.largeWidth = img.width;
        this.largeHeight = img.height;
      } else {
        img.onload = () => {
          this.largeWidth = img.width;
          this.largeHeight = img.height;
        };
      }
    },
    getImgProp(w, h) {
      this.ProportionX = this.largeWidth / w;
      this.ProportionY = this.largeHeight / h;
    },
    getCursorPos: function (e) {
      var x = window.Event
        ? e.pageX
        : event.clientX +
          (document.documentElement.scrollLeft
            ? document.documentElement.scrollLeft
            : document.body.scrollLeft);
      var y = window.Event
        ? e.pageY
        : event.clientY +
          (document.documentElement.scrollTop
            ? document.documentElement.scrollTop
            : document.body.scrollTop);
      this.cursorX = x - this.thumbPos.x;
      this.cursorY = y - this.thumbPos.y;
    },
    getBounds: function () {
      var el = this.$refs.magnificationElement;
      this.bounds = el.getBoundingClientRect();
      this.getImgProp(this.bounds.width, this.bounds.height);
      var xPos = 0;
      var yPos = 0;
      while (el) {
        var transform = this.getTransform(el);
        if (el.tagName === "BODY") {
          // deal with browser quirks with body/window/document and page scroll
          var xScroll = el.scrollLeft || document.documentElement.scrollLeft;
          var yScroll = el.scrollTop || document.documentElement.scrollTop;
          xPos +=
            el.offsetLeft - xScroll + el.clientLeft + parseInt(transform[0]);
          yPos +=
            el.offsetTop - yScroll + el.clientTop + parseInt(transform[1]);
        } else {
          // for all other non-BODY elements
          xPos +=
            el.offsetLeft -
            el.scrollLeft +
            el.clientLeft +
            parseInt(transform[0]);
          yPos +=
            el.offsetTop - el.scrollTop + el.clientTop + parseInt(transform[1]);
        }
        el = el.offsetParent;
      }
      this.thumbPos = {
        x: xPos,
        y: yPos,
      };
    },
    moveMagnifier: function (e) {
      e.preventDefault();
      this.getBounds();
      this.getCursorPos(e);
      this.backgroundPos = `${
        this.cursorX * this.ProportionX * -1 + this.options.width / 2
      }px ${this.cursorY * this.ProportionY * -1 + this.options.height / 2}px`;
    },
    getTransform: function (el) {
      var transform = window
        .getComputedStyle(el, null)
        .getPropertyValue("-webkit-transform");
      function rotate_degree(matrix) {
        var angle;
        if (matrix !== "none") {
          var values = matrix.split("(")[1].split(")")[0].split(",");
          var a = values[0];
          var b = values[1];
          angle = Math.round(Math.atan2(b, a) * (180 / Math.PI));
        } else {
          angle = 0;
        }
        return angle < 0 ? (angle += 360) : angle;
      }
      var results = transform.match(
        /matrix(?:(3d)\(-{0,1}\d+\.?\d*(?:, -{0,1}\d+\.?\d*)*(?:, (-{0,1}\d+\.?\d*))(?:, (-{0,1}\d+\.?\d*))(?:, (-{0,1}\d+\.?\d*)), -{0,1}\d+\.?\d*\)|\(-{0,1}\d+\.?\d*(?:, -{0,1}\d+\.?\d*)*(?:, (-{0,1}\d+\.?\d*))(?:, (-{0,1}\d+\.?\d*))\))/
      );
      var output = [0, 0, 0];
      if (results) {
        if (results[1] === "3d") {
          output = results.slice(2, 5);
        } else {
          results.push(0);
          output = results.slice(5, 9); // returns the [X,Y,Z,1] value;
        }
        output.push(rotate_degree(transform));
      }
      return output;
    },
  },
};
</script>
```

### css

```css
<style scoped lang="scss">
// Magnifying glass options
$border-size: 2px; // Modify the border width of the magnifying glass component
$border-color: #666666; // Modify the border color of the magnifying glass component
$magnifier-width: 200px; // Modify the width of the magnifying glass component
$magnifier-height: 200px; // Modify the height of the magnifying glass component
// Define your responsive sizes of
$sizes: (
  "(max-width: 320px)" 250px 250px,
  "(max-width: 480px)" 350px 350px,
  "(min-width: 481px)" 450px 450px,
  "(min-width: 1024px)" 550px 550px,
  "(min-width: 1280px)" 600px 600px
);
.magnifier-container {
  position: relative;
  height: 100%;
  .preview {
    position: relative;
    height: 100%;
    background: {
      repeat: no-repeat;
      size: 100% 100%;
    }
    display: block;
    clear: both;
    margin: 0 auto;
    cursor: none;
    .magnifying-glass {
      position: absolute;
      cursor: none;
      background: #fff no-repeat;
      display: none;
      pointer-events: none;
    }
    &:hover {
      .magnifying-glass {
        display: block;
      }
    }
  }
}
</style>
```

## API

| 属性        | 说明                        | 类型   | 默认值   |
| ----------- | --------------------------- | ------ | -------- |
| src         | 缩略图地址                  | String | -        |
| srcLarge    | 高清图地址                  | String | -        |
| width       | 放大镜宽度                  | Number | 200      |
| height      | 放大镜高度                  | Number | 200      |
| borderSize  | 放大镜边框尺寸              | Number | 2        |
| borderColor | 放大镜边框颜色              | String | #666     |
| shape       | 放大镜形状(circular/square) | String | circular |

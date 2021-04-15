<!--
 * @Author: your name
 * @Date: 2021-04-15 10:48:59
 * @LastEditTime: 2021-04-15 18:41:09
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \image-captcha\src\components\imgCode.vue
-->
<template>
  <div class="img-canvas">
    <canvas
      ref="mycanvas"
      :width="canvasWidth"
      :height="canvasHeight"
      @click="changeCode"
      style="border: 1px solid #d3d3d3"
    >
    </canvas>
  </div>
</template>

<script>
import md5 from 'md5';
import { defineComponent, toRefs, onMounted, ref, watch } from 'vue';
export default defineComponent({
  name: 'imgCode',
  props: {
    // 输入的验证码
    mdCode: {
      type: String,
      default: ''
    },
    ischange:{
      type: Boolean,
      default: false
    },
    // 验证码宽度
    canvasWidth: {
      type: Number,
      default: 112
    },
    // 验证码高度
    canvasHeight: {
      type: Number,
      default: 40
    }
  },
  emits: ['getResult'], //返回加密后的验证码的函数
  setup(props, context) {
    // 默认值设置
    const defaultData = {
      identifyCode: 'dk47', // 验证码值，未加密的
      identifyCodes: '1234567890abcdefghijklmnopqrstuvwxyz', // 生成验证码的元素，数字和字母
      fontSizeMin: 20, // 图片上验证文字的最小值
      fontSizeMax: 40, // 图片上验证文字的最大值
      backgroundColorMin: 180, // 图片背景色值最小
      backgroundColorMax: 240, // 图片背景色值最大
      colorMin: 50, // 文字色值最小
      colorMax: 160, // 文字色值最大
      lineColorMin: 40, // 干扰线色值最小
      lineColorMax: 120, // 干扰线色值最大
      dotColorMin: 0, // 干扰点色值最小
      dotColorMax: 255, // 干扰点色值最大
      lineSum: 4, // 干扰线数量
      dotSum: 40 // 干扰点数量
    };
    // 获取父组件传的数据，具有响应式
    const { canvasWidth, canvasHeight, mdCode,ischange } = toRefs(props);

    const mycanvas = ref(null);

    // 获取随机数
    const getRandomNum = (min, max) => {
      return Math.floor(Math.random() * (max - min) + min);
    };
    // 获取随机颜色
    const getRandomColor = (min, max) => {
      let r = getRandomNum(min, max);
      let g = getRandomNum(min, max);
      let b = getRandomNum(min, max);
      return `rgb(${r},${g},${b})`;
    };

    // 创建验证码图形
    const drawPic = () => {
      // 获取dom元素
      let canvas = mycanvas.value;
      // 创建绘图环境
      let ctx = canvas.getContext('2d');
      // 设置当前文本基线
      ctx.textBaseline = 'bottom';
      ctx.fillStyle = getRandomColor(
        defaultData.backgroundColorMin,
        defaultData.backgroundColorMax
      );
      ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value);
      for (let i = 0; i < defaultData.identifyCode.length; i++) {
        drawWord(ctx, defaultData.identifyCode[i], i);
      }
      drawLine(ctx);
      drawPoint(ctx);
    };

    // 绘制文字
    const drawWord = (ctx, txt, index) => {
      ctx.fillStyle = getRandomColor(
        defaultData.colorMin,
        defaultData.colorMax
      );
      ctx.font =
        getRandomNum(defaultData.fontSizeMin, defaultData.fontSizeMax) +
        'px SimHei';
      // x轴偏移值
      let offsetX =
        (index + 1) *
        (canvasWidth.value / (defaultData.identifyCode.length + 1));
      let offsetY = getRandomNum(
        defaultData.fontSizeMax,
        canvasHeight.value - 5
      );
      // console.log(offsetY)
      let deg = getRandomNum(-135, 135);
      ctx.translate(offsetX, offsetY);
      // // 修改坐标原点和旋转角度
      ctx.rotate(deg / 180);
      ctx.fillText(txt, 0, 0);
      // // 还原坐标原点和旋转角度
      ctx.rotate(-deg / 180);
      ctx.translate(-offsetX, -offsetY);
    };

    // 绘制干扰线
    const drawLine = (ctx) => {
      for (let i = 0; i < defaultData.identifyCode.length; i++) {
        ctx.strokeStyle = getRandomColor(
          defaultData.lineColorMin,
          defaultData.lineColorMax
        );
        ctx.beginPath();
        ctx.moveTo(
          getRandomNum(0, canvasWidth.value),
          getRandomNum(0, canvasHeight.value)
        );
        ctx.lineTo(
          getRandomNum(0, canvasWidth.value),
          getRandomNum(0, canvasHeight.value)
        );
        ctx.stroke();
      }
    };

    // 绘制干扰点
    const drawPoint = (ctx) => {
      for (let i = 0; i < 50; i++) {
        ctx.fillStyle = getRandomColor(0, 255);
        ctx.beginPath();
        ctx.arc(
          getRandomNum(0, canvasWidth.value),
          getRandomNum(0, canvasHeight.value),
          1,
          0,
          2 * Math.PI
        );
        ctx.fill();
      }
    };

    // 生成随机验证码
    const getRandomCode = () => {
      defaultData.identifyCode = '';
      for (let i = 0; i < 4; i++) {
        defaultData.identifyCode +=
          defaultData.identifyCodes[
            getRandomNum(0, defaultData.identifyCodes.length)
          ];
      }
      console.log(defaultData.identifyCode)
      // 绘制图片
      drawPic();
    };
    onMounted(() => {
      getRandomCode();
    });

    // 监听code变化
    watch(ischange, () => {
      // 验证
      console.log(mdCode)
      context.emit('getResult', md5(defaultData.identifyCode) === mdCode.value);
    });

    return {
      mycanvas,
      getRandomCode
    };
  },
  methods: {
    changeCode() {
      this.getRandomCode()
    }
  }
});
</script>
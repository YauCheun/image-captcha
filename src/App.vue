<!--
 * @Author: your name
 * @Date: 2021-04-15 10:43:53
 * @LastEditTime: 2021-04-15 18:41:39
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \image-captcha\src\App.vue
-->

<template>
  <img-code :mdCode="mdCode" :ischange="ischange" @getResult="getResult"></img-code>
  <input type="text" v-model="code" />
  <button @click="submit">确定</button>
</template>

<script>
import md5 from 'md5'
import imgCode from './components/imgCode.vue';
export default {
  components: {
    imgCode
  },
  data() {
    return {
      code: '',
      mdCode:'',
      ischange: false
    };
  },
  watch:{
    code(val){
      this.mdCode=md5(val)
    }
  },
  methods: {
    getResult(val) {
      console.log(val);
      // 验证不成功清空
      if(!val){
        this.mdCode=''
        console.log('验证不成功')
      }else{
        console.log('验证成功')
      }
    },
    submit(){
      this.ischange=!this.ischange
    }
  }
};
</script>

<style>
#app {
  height: 500px;
  width: 100%;
}
</style>
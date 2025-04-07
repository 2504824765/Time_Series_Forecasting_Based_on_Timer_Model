<script setup lang="ts">
import ItemWrap from "@/components/item-wrap";
import LeftTop from "./left-top.vue";
import LeftCenter from "./left-center.vue";
import LeftBottom from "./left-bottom.vue";
import CenterMap from "./center-map.vue";
import CenterModel from "@/views/index/center-model/index.vue"
import CenterBottom from "./center-bottom.vue";
import RightTop from "./right-top.vue";
import RightCenter from "./right-center.vue";
import RightBottom from "./right-bottom.vue";
import { ref } from 'vue'; 

const excelData = ref<{
  headers: string[];
  data: any[];
} | null>(null);

const handleFileLoaded = (data: { headers: string[]; data: any[] }) => {
  excelData.value = data;
};
</script>

<template>
  <div class="index-box">
    <div class="contetn_left">
      <!-- 左上 -->
      <div class="content_left_top">
        <!-- 左上左 -->
        <ItemWrap class="contetn_left-top content_left_top-item" title="Excel数据导入">
          <LeftTop @file-loaded="handleFileLoaded" />
        </ItemWrap>
        <ItemWrap class="contetn_left-top content_left_top-item" title="Excel数据导入">
          <LeftTop @file-loaded="handleFileLoaded" />
        </ItemWrap>
      </div>
      <!-- 左中 -->
      <ItemWrap class="contetn_left-center content_left_bottom-item" title="数据可视化">
        <LeftCenter :chart-data="excelData" />
      </ItemWrap>
      <!-- 左下 -->
      <!-- <ItemWrap
        class="contetn_left-bottom contetn_lr-item"
        title="传感器设备列表"
        style="padding: 0 10px 16px 10px"
      >
        <LeftBottom />
      </ItemWrap> -->
    </div>
    <div class="contetn_center">
      <!-- 中上 -->
      <CenterModel class="contetn_center_top content_center_top-item" title="传感器分布图" />
      <!-- 中下 -->
       <div class="content_center_bottom">
        <ItemWrap class="contetn_center-bottom" title="风险演化分析">
          <!-- <CenterBottom /> -->
          <RightTop :chart-data="excelData" />
        </ItemWrap>
        <ItemWrap class="contetn_center-bottom" title="风险演化分析">
          <!-- <CenterBottom /> -->
          <RightTop :chart-data="excelData" />
        </ItemWrap>
      </div>
    </div>
    <!-- <div class="contetn_right"> -->
      <!-- 右上 -->
      <!-- <ItemWrap class="contetn_left-bottom content_right_top-item" title="选取特征因变量">
        
      </ItemWrap> -->
      <!-- 右中 -->
      <!-- <ItemWrap
        class="contetn_left-bottom content_right_center-item" title="选取特征因变量">
        <RightTop :chart-data="excelData" />
      </ItemWrap> -->
      <!-- 右下 -->
      <!-- <ItemWrap class="contetn_left-bottom contetn_lr-item" title="执行器设备列表 ">
        <RightBottom />
      </ItemWrap> -->
    <!-- </div> -->
  </div>
</template>

<style scoped lang="scss">
.index-box {
  // 调整中间组件宽度
  width: 100%;
  // width: 300px;
  display: flex;
  min-height: calc(100% - 64px);
  justify-content: space-between;
}
//左边 右边 结构一样
.contetn_left,
.contetn_right {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  position: relative;
  // width: 540px;
  width: 50%; /* 调整左边宽度 */
  box-sizing: border-box;
  flex-shrink: 0;
}

.content_left_top {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  position: relative;
  // width: 540px;
  width: 100%; /* 调整左边宽度 */
  box-sizing: border-box;
  flex-shrink: 0;
}

.contetn_center {
  flex: 1;
  // width: 540px;
  width: 100px;
  margin: 0 54px;
  display: flex;
  flex-direction: column;
  // justify-content: space-around;
  .contetn_center-bottom {
    // height: 315px;
    height: 480PX;
  }
}

.content_center_bottom {
  flex: 1;
  // width: 540px;
  width: 100%;
  // margin: 0 54px;
  display: flex;
  flex-direction: row;
  // justify-content: space-around;
  .contetn_center-bottom {
    // height: 315px;
    height: 480PX;
  }
}

.contetn_lr-item {
  height: 310px;
}

.content_left_top-item {
  height: 400px;
}

.content_left_bottom-item {
  height: 500px;
}

.content_right_center-item {
  height: 500px;
}

.content_center_top-item {
  // height: 100px;
}

.content_right_top-item {
  height: 500px;
  // width: 100%;
}
</style>

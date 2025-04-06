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
      <ItemWrap class="contetn_left-top content_left_top-item" title="Excel数据导入">
        <LeftTop @file-loaded="handleFileLoaded" />
      </ItemWrap>
      <ItemWrap class="contetn_left-center content_left_bottom-item" title="数据可视化">
        <LeftCenter :chart-data="excelData" />
      </ItemWrap>
      <!-- <ItemWrap
        class="contetn_left-bottom contetn_lr-item"
        title="传感器设备列表"
        style="padding: 0 10px 16px 10px"
      >
        <LeftBottom />
      </ItemWrap> -->
    </div>
    <div class="contetn_center">
      <!--将中间的地图替换为3d模型-->
<!--      <CenterMap class="contetn_center_top" title="传感器分布图" />-->
      <CenterModel class="contetn_center_top content_center_top-item" title="传感器分布图" />
      <ItemWrap class="contetn_center-bottom" title="风险演化分析">
        <CenterBottom />
      </ItemWrap>
    </div>
    <div class="contetn_right">
      <CenterModel class="contetn_center_top content_right_top-item" title="传感器分布图" />
      <!-- <ItemWrap class="contetn_left-bottom content_right_top-item" title="选取特征因变量">
        
      </ItemWrap> -->
      <ItemWrap
        class="contetn_left-bottom content_right_center-item" title="选取特征因变量">
        <!-- <RightCenter /> -->
        <RightTop :chart-data="excelData" />
      </ItemWrap>
      <!-- <ItemWrap class="contetn_left-bottom contetn_lr-item" title="执行器设备列表 ">
        <RightBottom />
      </ItemWrap> -->
    </div>
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
  width: 540px;
  box-sizing: border-box;
  flex-shrink: 0;
}
.contetn_center {
  flex: 1;
  width: 540px;
  margin: 0 54px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  .contetn_center-bottom {
    height: 315px;
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

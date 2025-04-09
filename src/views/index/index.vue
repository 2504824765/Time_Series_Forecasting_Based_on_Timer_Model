<script setup lang="ts">
import ItemWrap from "@/components/item-wrap";
import LeftTop from "./left-top.vue";
import leftTopRight from "./left-top-right.vue";
import LeftCenter from "./left-center.vue";
import LeftBottom from "./left-bottom.vue";
import leftBottomRight from "./left-bottom-right.vue";
import CenterMap from "./center-map.vue";
import CenterModel from "@/views/index/center-model/index.vue"
import RightTop from "./right-top.vue";
import RightCenter from "./right-center.vue";
import RightBottom from "./right-bottom.vue";
import { ref } from 'vue'; 
import CenterBottom from "./center-bottom.vue";
import LeftBottomRight from "./left-bottom-right.vue";
import LeftTopRight from "./left-top-right.vue";

// 旧的单文件数据
const excelData = ref<{
  headers: string[];
  data: any[];
} | null>(null);

const fileNames = ref<string[]>([]);
const fileDataMap = ref<Record<string, any[]>>({});

const handleFolderData = (payload: { fileNames: string[], fileDataMap: Record<string, any[]> }) => {
  fileNames.value = payload.fileNames;
  fileDataMap.value = payload.fileDataMap;
};

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
        <ItemWrap class="contetn_left-top content_left_top-item" title="Excel文件夹导入">
          <LeftTop @folder-data-loaded="handleFolderData" />
        </ItemWrap>
        <!-- 左上右 -->
        <ItemWrap class="contetn_left-top content_left_top-item" title="可选特征">
          <!-- <CenterBottom :file-names="fileNames" :file-data-map="fileDataMap" /> -->
          <LeftTopRight :file-names="fileNames" />
        </ItemWrap>
      </div>

      <!-- 左中 -->
      <ItemWrap class="contetn_left-center content_left_bottom-item" title="选定特征可视化">
        <!-- <LeftCenter :chart-data="excelData" /> -->
        <!-- <LeftBottom :file-names="fileNames" :file-data-map="fileDataMap" /> -->
        <CenterBottom :file-names="fileNames" :file-data-map="fileDataMap" />
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
      <CenterModel class="contetn_center_top content_center_top-item" title="发动机示意图" />
      <!-- 中下 -->
      <div class="content_center_bottom">
        <ItemWrap class="contetn_center-bottom" title="Reconstruction Loss">
          <!-- <RightTop :chart-data="excelData" /> -->
          <LeftBottom :file-names="fileNames" :file-data-map="fileDataMap" />
          <!-- <CenterBottom :file-names="fileNames" :file-data-map="fileDataMap" /> -->
        </ItemWrap>
        <ItemWrap class="contetn_center-bottom" title="Siamese Loss">
          <!-- <RightTop :chart-data="excelData" /> -->
          <!-- <CenterBottom :file-names="fileNames" :file-data-map="fileDataMap" /> -->
          <LeftBottomRight :file-names="fileNames" :file-data-map="fileDataMap" />
        </ItemWrap>
      </div>
    </div>
  </div>
</template>


<style scoped lang="scss">
.index-box {
  width: 100%;
  display: flex;
  min-height: calc(100% - 64px);
  justify-content: space-between;
  height: 90%;
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
  margin-left: 50px;
  display: flex;
  flex-direction: column;
  // justify-content: space-around;
  .contetn_center-bottom {
    height: 315px;
    // height: 480PX;
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
    height: 395px;
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

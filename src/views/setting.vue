<script setup lang="ts">
import { useSettingStore } from "@/stores/index";
import { ref } from "vue";
import {storeToRefs} from "pinia"
const isScaleRadio = ref(false);
const leftBottomRadio=ref(true)
const rightBottomRadio=ref(true)
const modelRotate=ref(true) // 模型是否旋转
const model=ref(true) // 模型是否旋转
const settingStore = useSettingStore();
const {indexConfig}=storeToRefs(settingStore)

const init = () => {
  settingStore.initSetting();
  isScaleRadio.value = settingStore.isScale;

  leftBottomRadio.value=indexConfig.value.leftBottomSwiper
  rightBottomRadio.value=indexConfig.value.rightBottomSwiper
  modelRotate.value=indexConfig.value.modelRotateSpeed

};
init();
const handleClose = () => {};

const cancelClick = () => {
  settingStore.setSettingShow(false);
};

const confirmClick = () => {};
const isScaleChange = (flag: boolean) => {
  settingStore.setIsScale(flag);
};
const radiochange = (blag: boolean) => {
  settingStore.setIsScale(blag);
  // this.$store.commit('setting/updateSwiper', { val, type })
};
const indexRadioChange=(flag: boolean)=>{
  settingStore.setIndexConfig({
    leftBottomSwiper: leftBottomRadio.value,//左轮播
    rightBottomSwiper:rightBottomRadio.value,//右下轮播
    modelRotateSpeed: modelRotate.value,//模型旋转速度
  });
}
</script>

<template>
  <el-drawer v-model="settingStore.settingShow" direction="rtl" size="360px">
    <template #header>
      <h2 class="setting-title">设置</h2>
    </template>
    <template #default>
      <div class="left_shu">全局设置</div>
      <div class="setting_item">
        <span class="setting_label">
          是否进行自动适配<span class="setting_label_tip"
            >(默认分辨率1920*1080)</span
          >:
        </span>
        <div class="setting_content">
          <el-radio-group v-model="isScaleRadio" @change="(flag)=>isScaleChange(flag as boolean)">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </div>
      </div>
      <div class="left_shu">实时监测</div>
      <div class="setting_item">
        <span class="setting_label">
          传感器设备自动轮询: <span class="setting_label_tip"></span>
        </span>
        <div class="setting_content">
          <el-radio-group
            v-model="leftBottomRadio"
            @change="(flag)=>indexRadioChange(flag as boolean)"
          >
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </div>
      </div>
      <div class="setting_item">
        <span class="setting_label"> 执行器设备列表轮播: </span>
        <div class="setting_content">
          <el-radio-group
            v-model="rightBottomRadio"
            @change="(flag)=>indexRadioChange(flag as boolean)"
          >
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </div>
      </div>
      <!--模型菜单-->
      <div class="left_shu">模型菜单</div>
      <div class="setting_item">
        <span class="setting_label">
          模型是否旋转: <span class="setting_label_tip"></span>
        </span>
        <div class="setting_content">
          <el-radio-group
              v-model="modelRotate"
              @change="(flag)=>indexRadioChange(flag as boolean)"
          >
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </div>
      </div>
      <div class="setting_item">
        <span class="setting_label">
          模型材质: <span class="setting_label_tip"></span>
        </span>
        <div class="setting_content">
          <el-radio-group
              v-model="modelRotate"
              @change="(flag)=>indexRadioChange(flag as boolean)"
          >
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </div>
      </div>
    </template>
    <!-- <template #footer>
      <div style="flex: auto">
        <el-button @click="cancelClick">取消</el-button>
        <el-button type="primary" @click="confirmClick">确定</el-button>
      </div>
    </template> -->
  </el-drawer>
</template>

<style scoped lang="scss">
.setting-title {
  font-size: 20px;
  color: #000;
  font-weight: 900;
  text-align: center;
  line-height: 1;
}
.left_shu {
  color: #000;
  font-weight: 900;
  position: relative;
  text-indent: 10px;
  padding: 16px 0 10px 0;
  line-height: 1;
  &::before {
    display: block;
    content: " ";
    height: 16px;
    width: 4px;
    border-radius: 2px;
    background: #0072ff;
    position: absolute;
    left: 0px;
  }
}
.setting_item {
  font-size: 14px;
  line-height: 1.5;

  // display: flex;
  .setting_label {
    color: #555454;
  }
  .setting_label_tip {
    font-size: 12px;
    color: #838282;
  }
}
</style>

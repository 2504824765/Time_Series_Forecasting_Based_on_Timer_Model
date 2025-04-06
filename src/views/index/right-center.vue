<script setup lang="ts">
import { ref, reactive, nextTick  } from "vue";
import CapsuleChart from "@/components/datav/capsule-chart";
import { ranking } from "@/api";
import { ElMessage } from "element-plus";

// 自定义的假数据
const msg = ref([
  {
    "value": 94.1,
    "name": "sensor1"
  },
  {
    "value": 92.5,
    "name": "sensor2"
  },
  {
    "value": 91.8,
    "name": "sensor3"
  },
  {
    "value": 75.2,
    "name": "sensor4"
  },
  {
    "value": 45.7,
    "name": "sensor5"
  },
  {
    "value": 41.7,
    "name": "sensor6"
  },
  {
    "value": 24.0,
    "name": "sensor7"
  },
  {
    "value": 20.2,
    "name": "sensor8"
  }
]);

const config = ref({
  showValue: true,
  unit: "%",
});
const data = ref([]);
const getData = () => {
  ranking()
    .then((res) => {
      console.log("右中--风险预测", res);
      if (res.success) {
        // data.value = res.data; // 连了socket后，这里使用socket的数据
        data.value = msg.value; // 这里使用我们的假数据
      } else {
        ElMessage({
          message: res.msg,
          type: "warning",
        });
      }
    })
    .catch((err) => {
      ElMessage.error(err);
    });
};

// 生成新数据的函数
const generateNewData = (currentData:any) => {
  return currentData.map(sensor => {
    // 随机增减最多1，并保留一位小数
    const newValue = parseFloat((sensor.value + (Math.random() - 0.5) * 2).toFixed(1));
    // 确保值在0到100之间，并保留一位小数
    const clampedValue = parseFloat(Math.min(100, Math.max(0, newValue)).toFixed(1));
    return {
      ...sensor,
      value: clampedValue
    };
  });
};

getData();

// 设置定时器，每5秒更新数据
const updateInterval = setInterval(() => {
  // 更新数据
  data.value = generateNewData(data.value);
}, 2000);

// 如果需要清理定时器（例如，在组件销毁时）
import { onUnmounted } from 'vue';
onUnmounted(() => {
  clearInterval(updateInterval);
});


// websocket实时数据
// const socket = new WebSocket('ws://127.0.0.1:9999/demo/rightMiddle/1');
// socket.onopen = function (event) {
//   console.log('右侧中间数据 WebSocket连接已打开');
// };
//
// socket.onmessage = async function (event) {
//   const startTime = performance.now(); // 记录开始时间
//   try {
//     const rightMiddleData = JSON.parse(event.data);
//     console.log("风险预测的数据:",rightMiddleData)
//     data.value = rightMiddleData;
//
//     // 等待DOM更新
//     await nextTick();
//
//     const endTime = performance.now(); // 记录结束时间
//     const duration = endTime - startTime;
//     console.log(`右中--风险预测--数据渲染耗时: ${duration.toFixed(2)} 毫秒`);
//
//   } catch (error) {
//     console.error('WebSocket数据解析错误:', error);
//   }
// };
//
// socket.onerror = function (error) {
//   // ElMessage.error('WebSocket连接错误');
//   console.error('WebSocket连接错误:', error);
// };
//
// socket.onclose = function (event) {
//   // ElMessage.error('WebSocket连接关闭');
//   console.log('WebSocket连接已关闭');
// };

</script>

<template>
  <div class="right_bottom">
    <CapsuleChart :config="config" style="width: 100%; height: 260px" :data="data" />
  </div>
</template>

<style scoped lang="scss">
.right_bottom {
  box-sizing: border-box;
  padding: 0 16px;
}
</style>

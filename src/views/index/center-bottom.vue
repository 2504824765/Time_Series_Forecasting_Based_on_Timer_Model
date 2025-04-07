<template>
  <div class="content">
    <select class="content_select" v-model="selectedFileName" @change="drawChart">
      <option v-for="name in fileNames" :key="name" :value="name">{{ name }}</option>
    </select>
    <div ref="chartRef" class="chart-container"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, onBeforeUnmount } from 'vue';
import * as echarts from 'echarts';
import { useResizeObserver } from '@vueuse/core';

const props = defineProps({
  fileNames: Array,
  fileDataMap: Object
});

const selectedFileName = ref('');
const chartRef = ref(null);
let chartInstance = null;

const initChart = () => {
  if (!chartRef.value) return;

  chartInstance = echarts.init(chartRef.value);

  // 使用 ResizeObserver 监听容器尺寸变化
  const { stop } = useResizeObserver(chartRef, () => {
    chartInstance?.resize();
  });

  onBeforeUnmount(() => {
    stop(); // 停止监听
    chartInstance?.dispose();
  });
};

const drawChart = () => {
  if (!selectedFileName.value) return;

  const data = props.fileDataMap[selectedFileName.value];
  if (!data || data.length === 0) return;

  // 提取表头
  const headers = Object.keys(data[0]);
  const labels = data.map((_, index) => index + 1);
  const values1 = data.map(row => parseFloat(Object.values(row)[1]) || 0);
  const values2 = data.map(row => parseFloat(Object.values(row)[2]) || 0);

  // 获取每个因变量的最小值和最大值
  const minValue1 = Math.min(...values1);
  const maxValue1 = Math.max(...values1);
  const minValue2 = Math.min(...values2);
  const maxValue2 = Math.max(...values2);

  const option = {
    tooltip: {
      trigger: 'axis',
      formatter: (params) => {
        return `
          ${headers[0]}: ${params[0].axisValue}<br/>
          ${headers[1]}: ${params[0].data}<br/>
          ${headers[2]}: ${params[1].data}
        `;
      }
    },
    grid: {
      top: 20,
      right: 20,
      bottom: 40,
      left: 60,
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: labels,
      name: headers[0], // 设置横坐标的名称为第一个表头
      axisLabel: {
        rotate: labels.length > 5 ? 30 : 0, // 数据多时旋转标签
        overflow: 'truncate',
        width: 80,
        color: '#ffffff'  // 字体颜色设置为白色
      },
      axisTick: {
        alignWithLabel: true
      }
    },
    yAxis: {
      type: 'value',
      name: headers[1], // 设置纵坐标的名称为第二个表头
      axisLine: {
        show: true
      },
      axisLabel: {
        color: '#ffffff'  // 字体颜色设置为白色
      },
      min: Math.min(minValue1, minValue2), // 设置最小值为两个因变量的最小值
      max: Math.max(maxValue1, maxValue2)  // 设置最大值为两个因变量的最大值
    },
    series: [
      {
        name: headers[1], // 使用第一个因变量的表头作为系列名称
        data: values1,
        type: 'line',
        smooth: true,
        symbol: 'circle',
        symbolSize: 8,
        itemStyle: {
          color: '#36a3f7'
        },
        lineStyle: {
          width: 3,
          color: '#36a3f7'
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(54, 163, 247, 0.3)' },
            { offset: 1, color: 'rgba(54, 163, 247, 0.1)' }
          ])
        }
      },
      {
        name: headers[2], // 使用第二个因变量的表头作为系列名称
        data: values2,
        type: 'line',
        smooth: true,
        symbol: 'circle',
        symbolSize: 8,
        itemStyle: {
          color: '#ff7f50'
        },
        lineStyle: {
          width: 3,
          color: '#ff7f50'
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(255, 127, 80, 0.3)' },
            { offset: 1, color: 'rgba(255, 127, 80, 0.1)' }
          ])
        }
      }
    ],
    dataZoom: [
      {
        type: 'inside',
        start: 0,
        end: 100
      },
      {
        start: 0,
        end: 100
      }
    ]
  };

  chartInstance.setOption(option);
  chartInstance.resize();
};

watch(() => selectedFileName.value, (newVal) => {
  if (newVal) drawChart();
}, { immediate: true });

onMounted(() => {
  initChart();
});
</script>

<style scoped lang="scss">
.content {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  margin-left: -45px;

  h3 {
    font-size: large;
    text-align: center;
  }

  // 选择框
  .content_select {
    color: black;
    width: 35%;
    font-size: 23px;
  }
}
.chart-container {
  width: 100%;
  height: 400px;
  max-height: 500px;
  min-height: 300px;
  overflow: hidden;
  transition: all 0.3s;
  margin: 0 auto; // 居中
}
</style>

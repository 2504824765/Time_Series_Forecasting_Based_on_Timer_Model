<template>
  <div class="content">
    <div v-if="!fileNames || fileNames.length === 0" class="empty-message">
        <br><br><br><br><br><br><br><br><br><br>
        <p class="placeHolder">请先上传模型导出结果</p>
      </div>
    <div ref="chartRef" class="chart-container"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, onBeforeUnmount } from 'vue';
import * as echarts from 'echarts';
import { useResizeObserver } from '@vueuse/core';

interface FileData {
  [key: string]: any[];
}

const props = defineProps({
  fileNames: {
    type: Array as () => string[],
    default: () => []
  },
  fileDataMap: {
    type: Object as () => FileData,
    default: () => ({})
  }
});

const selectedFileName = ref('');
const chartRef = ref<HTMLElement | null>(null);
let chartInstance: echarts.ECharts | null = null;

// 初始化图表
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
    chartInstance = null;
  });
};

// 绘制图表
const drawChart = () => {
  if (!chartInstance || !selectedFileName.value) return;

  // 指定文件为loss图（或使用 selectedFileName.value 来选择当前选中的文件）
  const data = props.fileDataMap['loss_log.xlsx'] || props.fileDataMap[selectedFileName.value];
  if (!data || data.length === 0) {
    // 清空图表显示
    chartInstance.clear();
    return;
  }

  // 提取表头
  const headers = Object.keys(data[0]);
  const labels = data.map((_, index) => index + 1);
  const values1 = data.map(row => parseFloat(Object.values(row)[1]) || 0);
  const values2 = data.map(row => parseFloat(Object.values(row)[2]) || 0);

  // 获取每个因变量的最小值和最大值，并四舍五入取四位小数
  const roundToFour = (num: number) => {
    return parseFloat(num.toFixed(4));
  };

  const roundToSeven = (num: number) => {
    return parseFloat(num.toFixed(7));
  };

  // 获取每个因变量的最小值和最大值
  const minValue1 = roundToFour(Math.min(...values1));
  const maxValue1 = roundToFour(Math.max(...values1));
  const minValue2 = roundToFour(Math.min(...values2));
  const maxValue2 = roundToFour(Math.max(...values2));

  const option = {
    tooltip: {
      trigger: 'axis',
      formatter: (params: any) => {
        return `
          ${headers[0]}: ${params[0].axisValue}<br/>
          ${headers[1]}: ${roundToSeven(params[0].data)}
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
      name: headers[0],
      axisLabel: {
        rotate: labels.length > 5 ? 30 : 0,
        overflow: 'truncate',
        width: 80,
        color: '#ffffff'
      },
      axisTick: {
        alignWithLabel: true
      }
    },
    yAxis: {
      type: 'value',
      name: headers[1],
      axisLine: {
        show: true
      },
      axisLabel: {
        color: '#ffffff'  // 字体颜色设置为白色
      },
      min: Math.min(minValue1, minValue2),
      // min: minValue1,
      max: Math.max(maxValue1, maxValue2)
      // max: maxValue1
    },
    series: [
      {
        name: headers[1],
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
      // {
      //   name: headers[2],
      //   data: values2,
      //   type: 'line',
      //   smooth: true,
      //   symbol: 'circle',
      //   symbolSize: 8,
      //   itemStyle: {
      //     color: '#ff7f50'
      //   },
      //   lineStyle: {
      //     width: 3,
      //     color: '#ff7f50'
      //   },
      //   areaStyle: {
      //     color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
      //       { offset: 0, color: 'rgba(255, 127, 80, 0.3)' },
      //       { offset: 1, color: 'rgba(255, 127, 80, 0.1)' }
      //     ])
      //   }
      // }
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

// 监听props变化，自动绘制图表
watch(() => [props.fileNames, props.fileDataMap], ([newNames, newMap]) => {
  if (newNames && newNames.length > 0 && newMap && Object.keys(newMap).length > 0) {
    // 自动选择第一个文件（或指定文件）
    selectedFileName.value = newNames.find(name => name === 'loss_log.xlsx') || newNames[0];
    
    // 确保图表已初始化
    if (!chartInstance) {
      initChart();
    }
    
    // 绘制图表
    drawChart();
  }
}, { immediate: true });

// 监听选择的文件名变化
watch(selectedFileName, (newVal) => {
  if (newVal && chartInstance) {
    drawChart();
  }
});

onMounted(() => {
  initChart();
  
  // 如果初始已有数据，立即绘制
  if (props.fileNames.length > 0 && Object.keys(props.fileDataMap).length > 0) {
    selectedFileName.value = props.fileNames.find(name => name === 'loss_log.xlsx') || props.fileNames[0];
    drawChart();
  }
});
</script>

<style scoped lang="scss">
.placeHolder {
    font-size: 25px;
    margin-left: 30px;
}

.content {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  width: 100%;
  height: 95%;
  margin-left: -15px;

  .content_select {
    color: black;
    width: 80%;
    margin: 10px 0;
    padding: 8px;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
  }
  
  .chart-container {
    width: 100%;
    height: 100%;
    min-height: 300px;
    max-height: 500px;
    overflow: hidden;
    transition: all 0.3s;
  }
}
</style>
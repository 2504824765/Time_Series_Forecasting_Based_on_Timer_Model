<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import * as echarts from 'echarts';
import { useResizeObserver } from '@vueuse/core'; // 更精确的尺寸监听

const props = defineProps<{
  chartData?: {
    headers: string[];
    data: any[];
  }
}>();

const chartRef = ref<HTMLElement>();
let chartInstance: echarts.ECharts | null = null;

// 初始化图表
const initChart = () => {
  if (!chartRef.value) return;
  
  chartInstance = echarts.init(chartRef.value);
  
  // 使用ResizeObserver监听容器尺寸变化
  const { stop } = useResizeObserver(chartRef, () => {
    chartInstance?.resize();
  });
  
  onBeforeUnmount(() => {
    stop(); // 停止监听
    chartInstance?.dispose();
  });
};

// 更新图表数据
const updateChart = () => {
  if (!chartInstance || !props.chartData || props.chartData.data.length === 0) {
    return;
  }

  const { headers, data } = props.chartData;
  
  // 动态计算下边距（根据数据量）
  const gridBottom = data.length > 10 ? 60 : 40;
  
  const option = {
    tooltip: {
      trigger: 'axis',
      formatter: (params: any[]) => {
        return `
          ${headers[0]}: ${params[0].axisValue}<br/>
          ${headers[1]}: ${params[0].data}
        `;
      }
    },
    grid: {
      top: 20,
      right: 20,
      bottom: gridBottom,
      left: 60,
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: data.map(row => row[headers[0]]),
      name: headers[0],
      axisLabel: {
        rotate: data.length > 5 ? 30 : 0, // 数据多时旋转标签
        overflow: 'truncate',
        width: 80
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
      }
    },
    series: [{
      name: headers[1],
      data: data.map(row => row[headers[1]]),
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
    }],
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

// 监听数据变化
watch(() => props.chartData, (newVal) => {
  if (newVal) updateChart();
}, { deep: true });

onMounted(() => {
  initChart();
  // 延迟执行以确保DOM已渲染
  setTimeout(updateChart, 100);
});
</script>

<template>
  <div class="left-bottom-container">
    <!-- <h3>数据可视化</h3> -->
    <div class="chart-status">
      <div v-if="!chartData" class="no-data-hint">
        <i class="icon-empty"></i>
        <p>请先上传Excel文件生成图表</p>
      </div>
      <div v-else class="chart-info">
        当前展示: <strong>{{ chartData.headers[0] }}</strong> (X轴) vs 
        <strong>{{ chartData.headers[1] }}</strong> (Y轴)
      </div>
    </div>
    <div class="chart-wrapper">
      <div 
        ref="chartRef" 
        class="chart-container"
        :class="{ 'has-data': chartData }"
      ></div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.left-bottom-container {
  padding: 16px;
  height: 97%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  // background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

  h3 {
    margin: 0 0 8px 0;
    font-size: 20px;
    color: #c8c8c8;
    font-weight: 600;
    height: 20px;
  }

  .chart-status {
    margin-bottom: 12px;
    font-size: 16px;
    height: 16px;

    .no-data-hint {
      display: flex;
      align-items: center;
      color: #999;
      
      i {
        margin-right: 6px;
        font-size: 16px;
      }
    }

    .chart-info {
      color: #c8c8c8;
      strong {
        color: #36a3f7;
      }
    }
  }

  .chart-wrapper {
    flex: 1;
    min-height: 0;
    width: 100%;
    position: relative;
    overflow: hidden;
    border-radius: 4px;
    // background: #f9fafc;
    // border: 1px solid #eee;
  }

  .chart-container {
    margin-left: -45px;
    width: 103%;
    height: 100%;
    min-height: 300px;
    transition: all 0.3s;

    &.has-data {
      // background: #fff;
    }
  }
}
</style>
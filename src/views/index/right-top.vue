<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { graphic } from "echarts/core";
import { ElSelect, ElOption } from "element-plus";

const props = defineProps<{
  chartData?: {
    headers: string[];
    data: any[];
  }
}>();

// 当前选择的因变量列（默认为第二列）
const selectedColumn = ref<string>("");

// 可选的因变量列（排除第一列）
const dependentColumns = computed(() => {
  return props.chartData?.headers.slice(1) || [];
});

// 自动设置默认选中第一个因变量列
watch(() => props.chartData, (newData) => {
  if (newData?.headers && newData.headers.length > 1) {
    selectedColumn.value = newData.headers[1];
  }
}, { immediate: true });

// 图表配置
const option = computed(() => {
  if (!props.chartData || !selectedColumn.value) return {};
  
  const xData = props.chartData.data.map(row => row[props.chartData.headers[0]]);
  const yData = props.chartData.data.map(row => row[selectedColumn.value]);
  const columnIndex = props.chartData.headers.indexOf(selectedColumn.value);

  // 根据列索引分配不同颜色
  const colors = [
    { line: 'rgba(252,144,16,.7)', area: 'rgba(252,144,16,.7)', marker: '#FC9010' },
    { line: 'rgba(9,202,243,.7)', area: 'rgba(9,202,243,.7)', marker: '#09CAF3' },
    { line: 'rgba(202,9,188,.7)', area: 'rgba(202,9,188,.7)', marker: '#CA09BC' }
  ];
  const color = colors[columnIndex % 3] || colors[0];

  return {
    xAxis: {
      type: 'category',
      data: xData,
      boundaryGap: false,
      name: props.chartData.headers[0],
      axisLabel: { 
        rotate: xData.length > 10 ? 30 : 0,
        color: '#dddddd'
      },
      nameTextStyle: {  // X轴标题样式
        color: '#dddddd', // 绿色标题
        fontSize: 14,
        padding: [10, 0, 0, 0] // 调整标题位置
      }
    },
    yAxis: {
      type: 'value', name: selectedColumn.value,
      axisLabel: {  // Y轴标签样式
        color: '#dddddd', // 蓝色标签
        fontSize: 12
      },
      nameTextStyle: {  // Y轴标题样式
        color: '#dddddd', // 紫色标题
        fontSize: 14,
        padding: [0, 0, 0, 10] // 调整标题位置
      }
    },
    tooltip: { trigger: 'axis' },
    grid: { left: '10%', right: '10%', bottom: '15%' },
    series: [{
      data: yData,
      type: 'line',
      smooth: true,
      name: selectedColumn.value,
      itemStyle: { color: color.line },
      areaStyle: {
        color: new graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: color.area },
          { offset: 1, color: color.area.replace('7)', '0)') }
        ])
      },
    }]
  };
});
</script>

<template>
  <div class="chart-container">
    <!-- 列选择器 -->
    <div class="column-selector" v-if="dependentColumns.length > 0">
      <span class="selector-label">选择因变量：</span>
      <ElSelect v-model="selectedColumn" size="small">
        <ElOption
          v-for="col in dependentColumns"
          :key="col"
          :label="col"
          :value="col"
        />
      </ElSelect>
    </div>

    <!-- 图表展示 -->
    <div class="chart-wrapper">
      <v-chart
        v-if="option && Object.keys(option).length > 0"
        :option="option"
        :autoresize="true"
        class="chart"
      />
      <div v-else class="no-data">
        请先上传包含数据的Excel文件
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.chart-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 16px;

  .column-selector {
    margin-bottom: 16px;
    display: flex;
    align-items: center;

    .selector-label {
      margin-right: 8px;
      font-size: 14px;
      color: #bbbbbb;
      width: 120px;
    }
  }

  .chart-wrapper {
    flex: 1;
    position: relative;

    .chart {
      width: 100%;
      height: 100%;
      min-height: 300px;
    }

    .no-data {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100%;
      color: #909399;
      font-size: 14px;
    }
  }
}

/* 响应式调整 */
@media (max-width: 768px) {
  .column-selector {
    flex-direction: column;
    align-items: flex-start;

    .selector-label {
      margin-bottom: 8px;
    }
  }
}
</style>
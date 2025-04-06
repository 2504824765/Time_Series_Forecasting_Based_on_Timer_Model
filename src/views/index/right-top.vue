<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import { alarmNum } from "@/api";
import { graphic } from "echarts/core";
import { ElMessage } from "element-plus";
import { useAvgStore } from "@/stores/avg";

// 使用avg中的数据
const { avg } = useAvgStore()

const option = ref({});

// 自定义假数据
const msg:any = ref({
  dateList: [
    '1', '2', '3', '4', '5', '6', '7', '8', '9', '10',
    '11', '12', '13', '14', '15', '16', '17', '18', '19', '20',
    '21', '22', '23', '24', '25', '26', '27', '28', '29', '30'
  ],
  numList: [1.9247957645947717e-05, 4.475956542300274e-06, -3.796610890515638e-05, 0.0, -0.00018639954555909054, -2.0901990464204596, -0.0004819578877496976, -1.5827202049091005, -0.0008243146023882126, 9.688464662052562, -0.00040839994867986756, 7.278535413122013, -0.001137050769518388, 2.807936886778993, -0.0006239580661859404, 7.52986927409118, -0.007934306216291737, 8.912713496019917, -0.01404346398184474, 94.68495099420173, -0.022584573952277493, 56.19329823961074, -0.013696817050057172, -9.001209408667899, -0.044382906905775066, -3.6648302157920627, -0.016955880443125803, 6.328948674688815, -0.026414309475127427, 3.7917808425844606],
  numList2: [4.254196805850982e-05, 1.2288601574275994e-05, -0.0008764607679714976, 0.0, -0.0014206507008758665, -2.850617107803675, -0.0016058871956736345, -2.5930125452467774, 0.0020181519587768267, 16.227175115247743, 0.0020354054123958728, 8.740428852774759, 0.001863762716868184, 12.226074637995973, 0.0016435164894322116, 12.658879312453402, -0.015972328298059318, 172.1946827040127, -0.12218351771458669, 174.99134916357016, -0.12783532814054652, 141.94460458927966, -0.11928395108160511, 61.41992619206786, -0.09670722087027156, 81.90131699296145, -0.10098275971782665, 24.523768552919098, -0.048599806659870244, 15.32008606231045],
  numList3: [2.8084240026569114e-05, 1.0322938161740008e-05, 0.00017433230186336189, 0.00015651085141903172, -0.0005326751334860585, -0.4859314730411021, -0.00046171607615343543, -0.5678649249767266, 0.0019360459462068708, 11.931019544703085, 0.0010113116852234177, 7.386872745949685, 0.0010704540430641963, 9.27514712106089, 0.0010940427553716408, 9.028277338089271, -0.014510772713343213, 154.89766201615575, -0.06415984245667898, 162.56447759299354, -0.05030214081278045, 158.6012054727932, -0.058484337168457225, 43.50633222687323, -0.05698095733400319, 51.88713354308167, -0.05528423545197469, 19.63275243490163, -0.022054868993879453, 10.794396214902642],
});
// numList: [436, 589, 689, 789, 657, 590],
//     numList2: [34, 94, 134, 234, 334, 234],
//     numList3: [34, 94, 134, 234, 334, 234],

// 使用状态管理中的数据
let intervalId;
let isUsingFirstThreeRows = true; // 状态变量，用于追踪当前使用的行
const updateData = () => {
  const rowStart = isUsingFirstThreeRows ? 0 : 3;
  const rowEnd = rowStart + 3;

  // 更新numList, numList2, numList3
  for (let i = 0; i < 3; i++) {
    msg.value[`numList${i > 0 ? i + 1 : ''}`] = avg[rowStart + i];
  }

  // 切换状态
  isUsingFirstThreeRows = !isUsingFirstThreeRows;

  // 更新表单中的数据
  setOption(msg.value.dateList, msg.value.numList, msg.value.numList2 ,msg.value.numList3);
};



// 随机变动数据
// const updateData = () => {
//   // 获取当前时间，并格式化为 "HH:mm" 格式
//   const now = new Date();
//   const newDate = now.toLocaleTimeString('default', { hour: '2-digit', minute: '2-digit' });
//
//   // 确保新时间比最后一个时间晚一分钟
//   const lastTime = new Date(`2024-08-03T${msg.value.dateList[msg.value.dateList.length - 1]}`);
//   if (now < lastTime) {
//     // 如果当前时间早于最后一个时间的一分钟后，则等待一分钟后再次尝试更新
//     setTimeout(updateData, 60000 - (now.getTime() - lastTime.getTime()));
//     return;
//   }
//
//   // 更新日期列表
//   msg.value.dateList.push(newDate);
//   msg.value.dateList.shift(); // 移除第一个元素以保持数组长度不变
//
//   // 生成新的功率数据，变化不超过±10%
//   const numList = msg.value.numList.map(num => Math.round(num * (1 + (Math.random() * 20 - 10) / 100)));
//   const numList2 = msg.value.numList2.map(num => Math.round(num * (1 + (Math.random() * 20 - 10) / 100)));
//   const numList3 = msg.value.numList2.map(num => Math.round(num * (1 + (Math.random() * 20 - 10) / 100)));
//
//   // 更新功率数据
//   msg.value.numList = numList;
//   msg.value.numList2 = numList2;
//   msg.value.numList3 = numList3;
//
//   setOption(msg.value.dateList, msg.value.numList, msg.value.numList2 ,msg.value.numList3);
//
//   // console.log("msg数据", msg.value);
// };

const getData = () => {


  alarmNum()
    .then((res) => {
      console.log("右上--实时监测信息 ", res);
      if (res.success) {
        // 使用假数据
        setOption(msg.value.dateList, msg.value.numList, msg.value.numList2, msg.value.numList2);
        // setOption(res.data.dateList, res.data.numList, res.data.numList2);
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
const setOption = (xData: any[], yData: any[], yData2: any[], yData3: any[]) => {
  option.value = {
    xAxis: {
      type: 'category',
      data: xData,
      boundaryGap: false, // 不留白，从原点开始
      name: '时间', // 添加横轴名称
      splitLine: {
        show: true,
        lineStyle: {
          color: 'rgba(31,99,163,.2)',
        },
      },
      axisLine: {
        lineStyle: {
          color: 'rgba(31,99,163,.1)',
        },
      },
      axisLabel: {
        color: '#7EB7FD',
        fontWeight: '500',
      },
    },
    yAxis: {
      type: 'value',
      name: '功率', // 添加纵轴名称
      splitLine: {
        show: true,
        lineStyle: {
          color: 'rgba(31,99,163,.2)',
        },
      },
      axisLine: {
        lineStyle: {
          color: 'rgba(31,99,163,.1)',
        },
      },
      axisLabel: {
        color: '#7EB7FD',
        fontWeight: '500',
      },
    },
    tooltip: {
      trigger: 'axis',
      backgroundColor: 'rgba(0,0,0,.6)',
      borderColor: 'rgba(147, 235, 248, .8)',
      textStyle: {
        color: '#FFF',
      },
    },
    grid: {
      show: true,
      left: '10px',
      right: '30px',
      bottom: '10px',
      top: '32px',
      containLabel: true,
      borderColor: '#1F63A3',
    },
    series: [
      {
        data: yData,
        type: 'line',
        smooth: true,
        symbol: 'none', //去除点
        name: '报警1功率',
        color: 'rgba(252,144,16,.7)',
        areaStyle: {
          color: new graphic.LinearGradient(
              0,
              0,
              0,
              1,
              [
                {
                  offset: 0,
                  color: 'rgba(252,144,16,.7)',
                },
                {
                  offset: 1,
                  color: 'rgba(252,144,16,.0)',
                },
              ],
              false
          ),
        },
        markPoint: {
          data: [
            {
              name: '最大值',
              type: 'max',
              valueDim: 'y',
              symbol: 'rect',
              symbolSize: [60, 26],
              symbolOffset: [0, -20],
              itemStyle: {
                color: 'rgba(0,0,0,0)',
              },
              label: {
                color: '#FC9010',
                backgroundColor: 'rgba(252,144,16,0.1)',
                borderRadius: 6,
                padding: [7, 14],
                borderWidth: 0.5,
                borderColor: 'rgba(252,144,16,.5)',
                formatter: '报警1：{c}',
              },
            },
            {
              name: '最大值',
              type: 'max',
              valueDim: 'y',
              symbol: 'circle',
              symbolSize: 6,
              itemStyle: {
                color: '#FC9010',
                shadowColor: '#FC9010',
                shadowBlur: 8,
              },
              label: {
                formatter: '',
              },
            },
          ],
        },
      },
      {
        data: yData2,
        type: 'line',
        smooth: true,
        symbol: 'none', //去除点
        name: '报警2功率',
        color: 'rgba(9,202,243,.7)',
        areaStyle: {
          color: new graphic.LinearGradient(
              0,
              0,
              0,
              1,
              [
                {
                  offset: 0,
                  color: 'rgba(9,202,243,.7)',
                },
                {
                  offset: 1,
                  color: 'rgba(9,202,243,.0)',
                },
              ],
              false
          ),
        },

        markPoint: {
          data: [
            {
              name: '最大值',
              type: 'max',
              valueDim: 'y',
              symbol: 'rect',
              symbolSize: [60, 26],
              symbolOffset: [0, -20],
              itemStyle: {
                color: 'rgba(0,0,0,0)',
              },
              label: {
                color: '#09CAF3',
                backgroundColor: 'rgba(9,202,243,0.1)',
                borderRadius: 6,
                borderColor: 'rgba(9,202,243,.5)',
                padding: [7, 14],
                formatter: '报警2：{c}',
                borderWidth: 0.5,
              },
            },
            {
              name: '最大值',
              type: 'max',
              valueDim: 'y',
              symbol: 'circle',
              symbolSize: 6,
              itemStyle: {
                color: '#09CAF3',
                shadowColor: '#09CAF3',
                shadowBlur: 8,
              },
              label: {
                formatter: '',
              },
            },
          ],
        },
      },
      {
        data: yData3,
        type: 'line',
        smooth: true,
        symbol: 'none', //去除点
        name: '报警3功率',
        color: 'rgba(202,9,188,.7)',
        areaStyle: {
          color: new graphic.LinearGradient(
              0,
              0,
              0,
              1,
              [
                {
                  offset: 0,
                  color: 'rgba(202,9,188,.7)',
                },
                {
                  offset: 1,
                  color: 'rgba(202,9,188,.0)',
                },
              ],
              false
          ),
        },

        markPoint: {
          data: [
            {
              name: '最大值',
              type: 'max',
              valueDim: 'y',
              symbol: 'rect',
              symbolSize: [60, 26],
              symbolOffset: [0, -20],
              itemStyle: {
                color: 'rgba(0,0,0,0)',
              },
              label: {
                color: '#CA09F3',
                backgroundColor: 'rgba(202,9,243,0.1)',
                borderRadius: 6,
                borderColor: 'rgba(202,9,243,.5)',
                padding: [7, 14],
                formatter: '报警2：{c}',
                borderWidth: 0.5,
              },
            },
            {
              name: '最大值',
              type: 'max',
              valueDim: 'y',
              symbol: 'circle',
              symbolSize: 6,
              itemStyle: {
                color: '#09CAF3',
                shadowColor: '#09CAF3',
                shadowBlur: 8,
              },
              label: {
                formatter: '',
              },
            },
          ],
        },
      },
    ],
  };
};
onMounted(() => {
  setOption(msg.value.dateList, msg.value.numList, msg.value.numList2);
  const timer = setInterval(updateData, 5000); // 每隔10秒更新数据
  // 清除定时器
  onUnmounted(() => clearInterval(timer));

  console.log("avg", avg)

  getData();
});
</script>

<template>
  <v-chart class="chart" :option="option" v-if="JSON.stringify(option) != '{}'" />
</template>

<style scoped lang="scss"></style>

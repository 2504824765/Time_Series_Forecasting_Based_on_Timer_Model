<script setup lang="ts">
import { ref, reactive, onMounted, nextTick, onUnmounted } from "vue";
import { installationPlan } from "@/api";
import { graphic } from "echarts/core";
import { ElMessage } from "element-plus";
import { useAvgStore } from "@/stores/avg";

// 使用avg中的数据
const { avg } = useAvgStore()

const option = ref({});

// 假数据
const msg:any = ref({
  "category": [
    '1', '2', '3', '4', '5', '6', '7', '8', '9', '10',
    '11', '12', '13', '14', '15', '16', '17', '18', '19', '20',
    '21', '22', '23', '24', '25', '26', '27', '28', '29', '30'
  ],
  "barData": [-7.196849507176421e-06, -6.692803095698216e-06, -0.0032307976673562654, 0.0, 0.00013859737367548694, -0.6847656609940238, 0.00020797509696463668, -0.5714425580681594, -0.0010571808604173856, -11.977136442717438, -0.0008378080159897456, -5.3201501394998285, -0.0007907056140121982, -12.356640111435576, -0.0009650481054031524, -7.142898458680985, 0.020826335999078207, -120.32558388370035, 0.1073206197931031, -93.85710880959964, 0.09636518956294488, -121.6133692436564, 0.09260309961260177, -30.087627106519548, 0.09139274024707404, -32.03088636138381, 0.07586384833902167, -11.849760977957045, 0.04155919655044631, -9.341004061178952],
  "lineData": [6.900951876520289e-06, 3.5186120981895877e-06, -0.0022499555737697987, -0.00013042570951585976, -1.667509938088734e-06, 0.19705513170398484, -2.5799605752683284e-07, 0.20286074143532976, 0.0006818918888783141, 2.066076890216695, 0.000507756678318313, 2.4676380487370153, 0.000593082005607285, 5.157913685888919, 0.0005889552546402368, 4.336701248493259, -0.0039307582685975525, 42.949255428032856, -0.037270030558880465, 34.14118880658754, -0.04700129910887356, 39.13701518388304, -0.028603511079553053, 24.271190332707665, -0.03980209097814208, 18.71777516026038, -0.030270250488988622, 5.019052120823901, -0.014025658454870754, 3.688959334595366],
  "rateData": [1.1904360476330546e-05, 6.217213746229004e-06, 0.0003683914923152533, 0.0, -8.055382726010876e-05, 0.43123830902629817, -6.713686847060049e-05, 0.16510402391646609, -0.00010907858178125754, 10.144672042637744, -3.330711529665123e-05, 2.2212169824109003, -0.00031224157628750046, 4.63361450896308, -0.00019684539189258128, 3.0792009029530636, 0.0008752449969273728, 59.1161353775908, -0.010680198397426352, 78.1203705021188, 0.03429598108729308, 87.94591307307039, 0.012994328040074679, 35.28825220460762, -0.0009098641246790838, 9.649241556553987, -0.00815058584287374, -0.07860511864929262, 0.004849096881473948, 9.848318881083914]

});

// const msg = ref({
//   "category": [
//     "08-01", "08-02", "08-03", "08-04", "08-05", "08-06",
//     "08-07", "08-08", "08-09", "08-10", "08-11", "08-12", "08-13",
//     "08-14", "08-15", "08-16", "08-17", "08-18", "08-19", "08-20",
//     "08-21", "08-22", "08-23", "08-24", "08-25", "08-26"
//   ],
//   "barData": [
//     89, 93, 82, 95, 54, 12, 72,
//     93, 13, 14, 54, 68, 30, 80,
//     23, 15, 27, 20, 61, 30, 81,
//     37, 79, 64, 64, 49, 36, 62,
//     88, 49, 57, 57
//   ],
//   "lineData": [
//     179, 170, 101, 173, 128, 104, 96, 176,
//     70, 111, 81, 155, 35, 149, 89, 111, 58,
//     78, 149, 53, 47, 121, 141, 70, 138, 95, 52,
//     95, 114, 69, 106, 120
//   ],
//   "rateData": [
//     "50", "55", "81", "55", "42",
//     "12", "75", "53", "19", "13",
//     "67", "44", "86", "54", "26",
//     "14", "47", "26", "41", "57",
//     "55", "31", "56", "91", "46",
//     "52", "69", "65", "77", "71",
//     "54", "48"
//   ]
// });

// 使用状态管理中的数据
let intervalId;
let currentIndex = 6; // 开始于第7行，因为索引是从0开始的
const updateData = () => {
  // 检查是否已经到达最后一组数据
  if (currentIndex >= 18) {
    currentIndex = 6; // 重置索引以重新开始循环
  }

  // 更新数据
  for (let i = 0; i < 3 && currentIndex + i < 18; i++) {
    msg.value[`barData`] = avg[currentIndex + i];
    msg.value[`lineData`] = avg[currentIndex + i + 1];
    msg.value[`rateData`] = avg[currentIndex + i + 2];
    console.log("msg数据", msg.value);
    currentIndex += 3;
    break; // 每次只处理一组三条数据
  }

  setOption(msg.value)
};

// 更新数据
// const updateData = () => {
//   // 获取当前时间，并格式化为 "MM-DD" 格式
//   const now = new Date();
//   const newDate = now.toLocaleDateString('default', { month: '2-digit', day: '2-digit' });
//
//   // 确保新时间比最后一个时间晚一天
//   const lastTime = new Date(`2024-08-${msg.value.category[msg.value.category.length - 1].replace('-', '')}`);
//   if (now < lastTime) {
//     // 如果当前时间早于最后一个时间的一天后，则等待一天后再尝试更新
//     setTimeout(updateData, 24 * 60 * 60 * 1000 - (now.getTime() - lastTime.getTime()));
//     return;
//   }
//
//   // 更新日期列表
//   msg.value.category.push(newDate);
//   msg.value.category.shift(); // 移除第一个元素以保持数组长度不变
//
//   // 生成新的数据，变化不超过±10%
//   const barData = msg.value.barData.map(num => Math.round(num * (1 + (Math.random() * 20 - 10) / 100)));
//   const lineData = msg.value.lineData.map(num => Math.round(num * (1 + (Math.random() * 20 - 10) / 100)));
//   const rateData = msg.value.rateData.map(num => Math.round(num * (1 + (Math.random() * 20 - 10) / 100)));
//
//   // 更新数据
//   msg.value.barData = barData;
//   msg.value.lineData = lineData;
//   msg.value.rateData = rateData;
//
//   // console.log("msg数据", msg.value);
// };

const getData = () => {
  installationPlan()
    .then((res) => {
      console.log("中下--风险演化分析", res);
      if (res.success) {
        // 使用假数据
        setOption(msg.value);
        // setOption(res.data);
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
const setOption = async (newData: any) => {
  option.value = {
    tooltip: {
      trigger: "axis",
      backgroundColor: "rgba(0,0,0,.6)",
      borderColor: "rgba(147, 235, 248, .8)",
      textStyle: {
        color: "#FFF",
      },
      formatter: function (params: any) {
        // 添加单位
        var result = params[0].name + "<br>";
        params.forEach(function (item: any) {
          if (item.value) {
            if (item.seriesName == "指标3") {
              result += item.marker + " " + item.seriesName + " : " + item.value + "%</br>";
            } else {
              result += item.marker + " " + item.seriesName + " : " + item.value + "个</br>";
            }
          } else {
            result += item.marker + " " + item.seriesName + " :  - </br>";
          }
        });
        return result;
      },
    },
    legend: {
      data: ["指标1", "指标2", "指标3"],
      textStyle: {
        color: "#B4B4B4",
      },
      top: "0",
    },
    grid: {
      left: "50px",
      right: "40px",
      bottom: "30px",
      top: "20px",
    },
    xAxis: {
      data: newData.category,
      axisLine: {
        lineStyle: {
          color: "#B4B4B4",
        },
      },
      axisTick: {
        show: false,
      },
    },
    yAxis: [
      {
        splitLine: { show: false },
        axisLine: {
          lineStyle: {
            color: "#B4B4B4",
          },
        },

        axisLabel: {
          formatter: "{value}",
        },
      }
      // {
      //   splitLine: { show: false },
      //   axisLine: {
      //     lineStyle: {
      //       color: "#B4B4B4",
      //     },
      //   },
      //   axisLabel: {
      //     formatter: "{value}% ",
      //   },
      // },
    ],
    series: [
      {
        name: "指标1",
        type: "bar",
        barWidth: 10,
        itemStyle: {
          borderRadius: 5,
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: "#956FD4" },
            { offset: 1, color: "#3EACE5" },
          ]),
        },
        data: newData.barData,
      },
      {
        name: "指标2",
        type: "bar",
        barGap: "-100%",
        barWidth: 10,
        itemStyle: {
          borderRadius: 5,
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: "rgba(156,107,211,0.8)" },
            { offset: 0.2, color: "rgba(156,107,211,0.5)" },
            { offset: 1, color: "rgba(156,107,211,0.2)" },
          ]),
        },
        z: -12,
        data: newData.lineData,
      },
      {
        name: "指标3",
        type: "line",
        smooth: true,
        showAllSymbol: true,
        symbol: "emptyCircle",
        symbolSize: 8,
        // yAxisIndex: 1,
        itemStyle: {
          color: "#F02FC2",
        },
        data: newData.rateData,
      },
    ],
  };
};
onMounted(() => {
  const timer = setInterval(updateData, 2000); // 每隔2秒更新数据
  // 清除定时器
  onUnmounted(() => clearInterval(timer));

  getData();
});
</script>

<template>
  <v-chart class="chart" :option="option" v-if="JSON.stringify(option) != '{}'" />
</template>

<style scoped lang="scss"></style>

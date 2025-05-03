<template>
  <div ref="chartRef" class="chart-container"></div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import * as echarts from "echarts";

const props = defineProps<{
  scores: {
    流利: number;
    词汇: number;
    发音: number;
    语法: number;
  };
}>();

const chartRef = ref<HTMLElement | null>(null);
let chart: echarts.ECharts | null = null;

const initChart = () => {
  if (!chartRef.value) return;

  chart = echarts.init(chartRef.value);
  updateChart();
};

const updateChart = () => {
  if (!chart) return;

  const option = {
    radar: {
      shape: "polygon",
      indicator: [
        { name: "流利", max: 9 },
        { name: "词汇", max: 9 },
        { name: "发音", max: 9 },
        { name: "语法", max: 9 },
      ],
      splitArea: {
        show: false,
      },
      axisLine: {
        lineStyle: {
          color: "#4a90e2",
        },
      },
      splitLine: {
        lineStyle: {
          color: "#2a2b4a",
        },
      },
    },
    series: [
      {
        type: "radar",
        data: [
          {
            value: [
              props.scores.流利,
              props.scores.词汇,
              props.scores.发音,
              props.scores.语法,
            ],
            areaStyle: {
              color: "rgba(74, 144, 226, 0.2)",
            },
            lineStyle: {
              color: "#4a90e2",
            },
          },
        ],
      },
    ],
  };

  chart.setOption(option);
};

watch(
  () => props.scores,
  () => {
    updateChart();
  },
  { deep: true }
);

onMounted(() => {
  initChart();
  window.addEventListener("resize", () => {
    chart?.resize();
  });
});
</script>

<style>
.chart-container {
  width: 300px;
  height: 300px;
}
</style>
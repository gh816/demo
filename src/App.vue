<template>
  <div class="report-container">
    <div class="header custom-header">
      <img
        src="@/assets/back.svg"
        alt="Back"
        class="back-icon"
        @click="goBack"
      />
      <span class="title-text">雅思口语评分报告</span>
      <img src="@/assets/Share.svg" alt="Share" class="share-icon" />
    </div>
    <div class="content">
      <div class="report-card">
        <div class="card-header">
          <div class="title-section">
            <div class="title">
              <span class="main-title">Movie</span>
              <span class="sub-title">Part1</span>
            </div>
            <div class="report-time">报告时间：{{ reportTime }}</div>
            <div class="powered-by">
              <span class="prefix-text">powered by</span>
              <span class="brand-name">破壳AI口语</span>
              <span class="lab-name">实验室</span>
            </div>
          </div>
          <div class="total-score-container">
            <div class="hexagon-container">
              <canvas id="hexagon-canvas" width="120" height="120"></canvas>
              <div class="score-display">
                <div class="band">B A N D</div>
                <div class="score">{{ averageScore.toFixed(1) }}</div>
                <div class="simulation-text">
                  Simulated by<br />Certified IELTS<br />Examiner
                </div>
              </div>
            </div>
            <div class="score-desc">
              分数说明 <el-icon><InfoFilled /></el-icon>
            </div>
          </div>
        </div>

        <!-- 雷达图容器 -->
        <div id="radar-chart" style="width: 100%; height: 300px"></div>
      </div>
      <div class="more-details">
        <!-- 更新链接文本 -->
        <el-link type="primary" :underline="false" @click="handleClick"
          >查看练习详情 ></el-link
        >
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed, onUnmounted } from "vue"; // 添加onUnmounted
import { InfoFilled } from "@element-plus/icons-vue"; // Removed Share
import * as echarts from "echarts/core";
import { RadarChart } from "echarts/charts";
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  LegendComponent,
  PolarComponent,
} from "echarts/components";
import { CanvasRenderer } from "echarts/renderers";

echarts.use([
  TitleComponent,
  TooltipComponent,
  GridComponent,
  RadarChart,
  CanvasRenderer,
  LegendComponent,
  PolarComponent,
]);

interface ScoreData {
  流利: number;
  词汇: number;
  语法: number;
  发音: number;
}

interface ApiResponse {
  reportTime: string;
  scores: ScoreData;
}

const reportTime = ref("");
const scores = ref<ScoreData>({
  流利: 0,
  词汇: 0,
  语法: 0,
  发音: 0,
});
const error = ref<string | null>(null);

// 模拟API响应数据结构
const mockApiResponse: ApiResponse = {
  reportTime: "2024-12-31 10:06",
  scores: {
    流利: 7.0,
    词汇: 6.0,
    语法: 6.0,
    发音: 5.0,
  },
};

const goBack = () => {
  console.log("Go back");
};
const handleClick = () => {
  console.log("跳转路由逻辑");
};

const averageScore = computed(() => {
  const scoreValues = Object.values(scores.value);
  if (scoreValues.length === 0) return 0;
  const sum = scoreValues.reduce((acc, score) => acc + score, 0);
  return sum / scoreValues.length;
});

// 存储echarts实例的引用，以便在组件卸载时销毁
let radarChart: echarts.ECharts | null = null;

// ECharts 初始化 和 Canvas 绘制
onMounted(() => {
  // 先加载数据
  reportTime.value = mockApiResponse.reportTime;
  scores.value = { ...mockApiResponse.scores };

  // 初始化 ECharts 雷达图
  const chartDom = document.getElementById("radar-chart");
  if (chartDom) {
    radarChart = echarts.init(chartDom);
    const option = {
      radar: {
        indicator: [
          { name: "词汇", max: 9.0 },
          { name: "流利", max: 9.0 },
          { name: "发音", max: 9.0 },
          { name: "语法", max: 9.0 },
        ],
        radius: "62%", // 调整雷达图大小
        center: ["50%", "55%"], // 雷达图位置
        shape: "circle", // 设置雷达图形状为圆形
        startAngle: 45, // 添加起始角度来旋转图表
        splitNumber: 4, // 设置为5个分割圈
        axisName: {
          color: "#ccc", // 指示器文字颜色调暗一些
          fontSize: 14,
          padding: [0, 0, 15, 0], // 增加底部padding，将标签往外推
          formatter: (name: string) => {
            const score = scores.value[name as keyof typeof scores.value];
            let scoreTag = "";

            // 根据不同指标设置不同颜色的分数标签
            if (name === "词汇") {
              scoreTag = "scorePurple";
            } else if (name === "流利") {
              scoreTag = "scoreTeal";
            } else if (name === "发音") {
              scoreTag = "scoreRed";
            } else if (name === "语法") {
              scoreTag = "scoreYellow";
            }

            // 返回富文本格式，先显示文字，然后是带圆圈的分数
            return ` {nameText|${name}} {${scoreTag}|${score.toFixed(1)}}`;
          },
          rich: {
            // 文字样式
            nameText: {
              color: "#ffffff",
              fontSize: 14,
              padding: [0, 5, 0, 0],
              align: "right",
            },
            // 分数样式 - 完美圆形标签
            scoreTeal: {
              backgroundColor: "#00bfa5",
              color: "#fff",
              borderRadius: 20,
              width: 32,
              height: 32,
              lineHeight: 32,
              padding: [0, 0],
              fontSize: 14,
              fontWeight: "bold",
              align: "center",
            },
            scorePurple: {
              backgroundColor: "#ab47bc",
              color: "#fff",
              borderRadius: 20,
              width: 32,
              height: 32,
              lineHeight: 32,
              padding: [0, 0],
              fontSize: 14,
              fontWeight: "bold",
              align: "center",
            },
            scoreYellow: {
              backgroundColor: "#ffab00",
              color: "#fff",
              borderRadius: 20,
              width: 32,
              height: 32,
              lineHeight: 32,
              padding: [0, 0],
              fontSize: 14,
              fontWeight: "bold",
              align: "center",
            },
            scoreRed: {
              backgroundColor: "#ff5252",
              color: "#fff",
              borderRadius: 20,
              width: 32,
              height: 32,
              lineHeight: 32,
              padding: [0, 0],
              fontSize: 14,
              fontWeight: "bold",
              align: "center",
            },
          },
        },
        splitArea: {
          // 分隔区域样式 - 模拟设计稿内部层次感
          areaStyle: {
            color: [
              "rgba(40, 42, 70, 0.3)", // 更深的区域
              "rgba(35, 37, 65, 0.3)", // 稍浅的区域
            ],
            shadowColor: "rgba(0, 0, 0, 0.2)",
            shadowBlur: 10,
          },
        },
        axisLine: {
          // 坐标轴线 - 更细更暗
          show: false, // 设置为 false 来隐藏轴线
          lineStyle: {
            color: "rgba(255, 255, 255, 0.2)",
            width: 1,
          },
        },
        splitLine: {
          // 分隔线 - 更细更暗
          lineStyle: {
            color: "rgba(255, 255, 255, 0.2)",
            width: 1,
          },
        },
      },
      series: [
        {
          name: "口语评分",
          type: "radar",
          data: [
            {
              value: [
                // 更新数据顺序以匹配新的 indicator 顺序
                scores.value.词汇, // 词汇分数在前
                scores.value.流利, // 流利分数在后
                scores.value.发音, // 发音分数
                scores.value.语法, // 语法分数
              ],
              name: "分数",
            },
          ],
          symbol: "circle", // 数据点标记形状
          symbolSize: 4, // 数据点标记大小 (减小)
          itemStyle: {
            // 数据点样式
            color: "#4dd0e1", // 数据点颜色 (浅蓝绿色)
          },
          lineStyle: {
            // 雷达图线条样式
            color: "#4dd0e1", // 线条颜色 (浅蓝绿色)
            width: 2,
          },
          areaStyle: {
            // 雷达图区域填充样式 - 使用更匹配的颜色和透明度
            color: "rgba(77, 208, 225, 0.5)", // 填充颜色及透明度 (浅蓝绿色，半透明)
          },
        },
      ],
    };
    radarChart.setOption(option);

    // 监听窗口大小变化，重新渲染图表
    window.addEventListener("resize", handleResize);
  } else {
    console.error("Radar chart container not found");
  }

  // 绘制六边形
  const canvas = document.getElementById("hexagon-canvas") as HTMLCanvasElement;
  if (canvas) {
    const ctx = canvas.getContext("2d");
    if (ctx) {
      const width = canvas.width;
      const height = canvas.height;
      const centerX = width / 2;
      const centerY = height / 2;
      const radius = Math.min(width, height) / 2 - 5; // 边缘留出一点空间

      // 清除之前的绘制内容
      ctx.clearRect(0, 0, width, height);

      // 绘制六边形
      ctx.beginPath();

      // 从顶部顶点开始，顺时针绘制六边形
      const startAngle = -Math.PI / 2; // 从正上方开始（-90度）
      const cornerRadius = 8; // 圆角大小，可以调整这个值来控制圆角程度

      // 计算六个顶点的坐标
      const vertices = [];
      for (let i = 0; i < 6; i++) {
        const currentAngle = startAngle + (i * Math.PI) / 3;
        vertices.push({
          x: centerX + radius * Math.cos(currentAngle),
          y: centerY + radius * Math.sin(currentAngle),
        });
      }

      // 绘制带圆角的六边形
      for (let i = 0; i < 6; i++) {
        const current = vertices[i];
        const next = vertices[(i + 1) % 6];
        const prev = vertices[(i - 1 + 6) % 6];

        // 计算当前顶点到前后顶点的方向向量
        const toPrev = { x: prev.x - current.x, y: prev.y - current.y };
        const toNext = { x: next.x - current.x, y: next.y - current.y };

        // 计算向量长度
        const toPrevLength = Math.sqrt(
          toPrev.x * toPrev.x + toPrev.y * toPrev.y
        );
        const toNextLength = Math.sqrt(
          toNext.x * toNext.x + toNext.y * toNext.y
        );

        // 计算圆角起点和终点（从顶点向内移动一定距离）
        const startPoint = {
          x: current.x + (toPrev.x / toPrevLength) * cornerRadius,
          y: current.y + (toPrev.y / toPrevLength) * cornerRadius,
        };

        const endPoint = {
          x: current.x + (toNext.x / toNextLength) * cornerRadius,
          y: current.y + (toNext.y / toNextLength) * cornerRadius,
        };

        // 第一个点，移动到起点
        if (i === 0) {
          ctx.moveTo(startPoint.x, startPoint.y);
        }

        // 绘制从上一个顶点到当前顶点的曲线
        ctx.lineTo(startPoint.x, startPoint.y);
        // 使用二次贝塞尔曲线绘制圆角
        ctx.quadraticCurveTo(current.x, current.y, endPoint.x, endPoint.y);
      }

      ctx.closePath();

      // 添加渐变效果
      const gradient = ctx.createLinearGradient(
        centerX - radius,
        centerY - radius,
        centerX + radius,
        centerY + radius
      );
      gradient.addColorStop(0, "#5BC7D8");
      gradient.addColorStop(1, "#4dd0e1");

      // 添加阴影效果
      // ctx.shadowColor = "rgba(91, 199, 216, 0.4)";
      ctx.shadowBlur = 10;
      ctx.shadowOffsetX = 0;
      ctx.shadowOffsetY = 0;

      // 添加背景填充色
      ctx.fillStyle = "#140E3C";
      ctx.fill();

      ctx.strokeStyle = gradient; // 使用渐变色描边
      ctx.lineWidth = 1; // 稍微加粗线条使其更清晰
      ctx.stroke();
    }
  }
});

// 处理窗口大小变化的函数
const handleResize = () => {
  if (radarChart) {
    radarChart.resize();
  }
};

onUnmounted(() => {
  window.removeEventListener("resize", handleResize);

  // 销毁ECharts实例
  if (radarChart) {
    radarChart.dispose();
    radarChart = null;
  }
});
</script>

<style>
body {
  margin: 0;
  background-color: #1a1b2f;
}

.report-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 15px;
  background-color: #1a1b2f;
  min-height: 100vh;
  color: white;
  box-sizing: border-box;
}

.header {
  margin-bottom: 20px;
  padding: 0 5px;
}

.custom-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 50px;
}

.back-icon,
.share-icon {
  width: 24px;
  height: 24px;
  cursor: pointer;
  color: white;
}

.title-text {
  color: white;
  font-size: 18px;
  font-weight: 500;
  text-align: center;
  flex-grow: 1;
}

.report-card {
  background: #34316e;
  border-radius: 15px;
  padding: 25px;
  position: relative;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 15px;
}

.title-section {
  display: flex;
  flex-direction: column;
}

.title {
  margin-bottom: 20px;
}

.main-title {
  font-size: 26px;
  font-weight: bold;
  margin-right: 10px;
  color: #ffffff;
}

.sub-title {
  background-color: #3de1fa;
  color: #2a2450;
  padding: 1px 5px;
  border-radius: 5px;
  font-size: 12px;
  font-weight: bold;
  vertical-align: middle;
}

.report-time {
  font-size: 14px;
  color: #a0a0a0;
  margin-bottom: 4px;
}

.powered-by {
  font-size: 12px;
  color: #888;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
}

.brand-name {
  color: #ffffff;
  font-size: 12px;
  margin: 0 7px;
}

.powered-by .logo {
  height: 16px;
  margin: 0 5px;
  vertical-align: middle;
}

.total-score-container {
  text-align: center;
  position: relative;
  margin-top: 5px;
}

.hexagon-container {
  position: relative;
  width: 120px;
  height: 120px;
  margin-bottom: 5px;
}

#hexagon-canvas {
  display: block;
}

.score-display {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.band {
  font-size: 12px;
  color: #ffffff;
  margin-bottom: 2px;
  font-size: 11px;
  font-weight: 12;
  font-family: "PingFang SC", sans-serif; /* Add font family */
}

.score {
  font-size: 25px;
  font-weight: bold;
  color: #ffffff;
  line-height: 1;
  margin-bottom: 4px;
}

.simulation-text {
  font-size: 9px;
  color: #e0e0e0;
  line-height: 1.1;
  text-align: center;
  margin-top: 6px;
  font-family: "PingFang SC", sans-serif;
  font-weight: 100;
}

.score-desc {
  font-size: 12px;
  color: #a0a0a0;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 8px;
}

.score-desc .el-icon {
  margin-left: 4px;
  color: #a0a0a0;
}

#radar-chart {
  margin-top: -10px;
  margin-bottom: 10px;
  background: url("@/assets/radar-bg.svg") no-repeat center center/contain;
  position: relative;
  background-color: #34316e;
  border-radius: 10px;
}

#radar-chart canvas {
  position: relative;
  z-index: 2;
}

.more-details {
  text-align: center;
  margin-top: 15px; /* 调整间距 */
}

.more-details .el-link {
  font-size: 14px;
  color: #a0a0a0 !important; /* 链接颜色 */
}
.more-details .el-link:hover {
  color: #ffffff !important; /* 悬停颜色 */
}

/* 响应式调整（可选） */
@media (max-width: 480px) {
  .report-container {
    padding: 10px;
  }
  .report-card {
    padding: 20px;
  }
  .main-title {
    font-size: 22px;
  }
  .score {
    font-size: 30px;
  }
  #radar-chart {
    height: 250px; /* 移动设备上减小图表高度 */
  }
}
</style>

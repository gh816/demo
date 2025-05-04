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
import { ref, onMounted, computed, onUnmounted, onBeforeUnmount } from "vue";
import { InfoFilled } from "@element-plus/icons-vue";
import * as echarts from "echarts/core";
import {
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  RadarComponent,
  GridComponent,
} from "echarts/components";
import { CanvasRenderer } from "echarts/renderers";
import * as d3 from "d3";

echarts.use([
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  RadarComponent,
  CanvasRenderer,
  GridComponent,
]);

// 定义接口
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

const reportTime = ref<string>("");
const scores = ref<ScoreData>({
  词汇: 6.0,
  流利: 7.0,
  发音: 5.0,
  语法: 6.0,
});
const error = ref<string | null>(null);

// 模拟API响应数据结构
const mockApiResponse: ApiResponse = {
  reportTime: "2024年8月2日",
  scores: {
    词汇: 6.0,
    流利: 7.0,
    发音: 5.0,
    语法: 6.0,
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

// 窗口大小变化处理函数
const handleResize = () => {
  if (radarChart) {
    radarChart.resize();
  }
  // 如果d3雷达图存在，也需要重新渲染
  if (document.getElementById("radar-chart")) {
    drawRadarChartWithD3();
  }
};

// ECharts 初始化 和 Canvas 绘制
onMounted(() => {
  // 先加载数据
  reportTime.value = mockApiResponse.reportTime;
  scores.value = { ...mockApiResponse.scores };

  // 使用d3绘制雷达图
  drawRadarChartWithD3();

  // 监听窗口大小变化，重新渲染图表
  window.addEventListener("resize", handleResize);

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

// 使用d3.js绘制雷达图
const drawRadarChartWithD3 = () => {
  // 清除之前的SVG
  d3.select("#radar-chart svg").remove();

  const chartDom = document.getElementById("radar-chart");
  if (!chartDom) return;

  // 设置图表尺寸和中心点
  const width = chartDom.clientWidth;
  const height = 300; // 根据你的div高度设置

  // 向左下方调整中心点
  const centerX = width / 2 - 5; // 向左移动5px
  const centerY = height / 2 - 15; // 现在只向上移动15px，比之前少移动一些，相当于向下调整

  const radius = (Math.min(width, height) / 2) * 0.62; // 与echarts一致的半径比例

  // 创建SVG
  const svg = d3
    .select("#radar-chart")
    .append("svg")
    .attr("width", width)
    .attr("height", height)
    .style("position", "absolute")
    .style("top", 0)
    .style("left", 0)
    .style("z-index", 2); // 确保在背景图之上

  // 数据和刻度 - 保持原始顺序，不改变菱形
  const maxScore = 9.0; // 最大分数
  const features = ["词汇", "流利", "发音", "语法"]; // 恢复原始顺序
  const angleSlice = (Math.PI * 2) / features.length;

  // 分数缩放函数
  const rScale = d3.scaleLinear().domain([0, maxScore]).range([0, radius]);

  // 构建雷达图数据点
  const radarData = features.map((feature, i) => {
    const value = scores.value[feature];
    const scaledValue = rScale(value);
    // 修正角度计算，从顶部开始，顺时针旋转，确保从正中心开始绘制
    // 添加45度的偏移
    const angle = angleSlice * i - Math.PI / 2 + Math.PI / 4;
    return {
      feature,
      value,
      x: centerX + scaledValue * Math.cos(angle),
      y: centerY + scaledValue * Math.sin(angle),
      angle, // 保存角度用于放置标签
    };
  });

  // 使用d3.line类型正确的方式定义连接线生成器
  const lineGenerator = d3
    .line<{ x: number; y: number }>()
    .x((d) => d.x)
    .y((d) => d.y)
    .curve(d3.curveLinearClosed);

  // 绘制雷达图连线和区域
  svg
    .append("path")
    .datum(radarData)
    .attr("class", "radar-line")
    .attr("d", lineGenerator as any) // 使用类型断言解决TypeScript错误
    .style("fill", "none") // 移除填充色，改为透明
    .style("stroke", "#4dd0e1") // 保留边框线条
    .style("stroke-width", "2px");

  // 标签位置数组 - 交换语法和流利的位置
  const labelPositions = [
    { feature: "词汇", angleIndex: 0 },
    { feature: "语法", angleIndex: 1 },
    { feature: "发音", angleIndex: 2 },
    { feature: "流利", angleIndex: 3 },
  ];

  // 添加四个评分标签 - 使用自定义位置
  labelPositions.forEach((labelPos) => {
    const feature = labelPos.feature;
    const value = scores.value[feature];
    const i = labelPos.angleIndex;

    // 计算标签位置，放在雷达图外围一定距离处
    const angle = angleSlice * i - Math.PI / 2 + Math.PI / 4;
    const labelRadius = radius * 1.3; // 标签放在雷达图外30%的位置
    const labelX = centerX + labelRadius * Math.cos(angle);
    const labelY = centerY + labelRadius * Math.sin(angle);

    // 创建标签组
    const label = svg
      .append("g")
      .attr("class", "score-label")
      .attr("transform", `translate(${labelX}, ${labelY})`);

    label
      .append("rect")
      .attr("width", 70) // 增加胶囊宽度
      .attr("height", 30) // 胶囊高度
      .attr("x", -35) // 水平居中
      .attr("y", -15) // 垂直居中
      .attr("rx", 15) // 水平圆角半径，等于高度的一半，形成胶囊形状
      .attr("ry", 15) // 垂直圆角半径
      .style("fill", "#41407f") // 背景色
      .style("opacity", 0.95) // 降低透明度，使其更加可见
      .style("stroke", "#4a477e") // 添加边框
      .style("stroke-width", "1px"); // 边框宽度

    // 根据指标类型选择颜色
    let bgColor;
    if (feature === "流利") {
      bgColor = "#00bfa5";
    } else if (feature === "词汇") {
      bgColor = "#ab47bc";
    } else if (feature === "发音") {
      bgColor = "#ff5252";
    } else if (feature === "语法") {
      bgColor = "#ffab00";
    }

    if (feature === "词汇" || feature === "语法") {
      // 数字的圆形背景放在左侧
      label
        .append("circle")
        .attr("r", 13) // 圆形背景半径为13，总宽度为26
        .attr("cx", -15)
        .attr("cy", 0)
        .style("fill", bgColor);

      // 添加分数值
      label
        .append("text")
        .attr("text-anchor", "middle")
        .attr("x", -15)
        .attr("y", 5)
        .attr("font-size", "14px")
        .attr("font-weight", "bold")
        .attr("fill", "#ffffff")
        .text(value.toFixed(1));

      // 添加右侧文本
      label
        .append("text")
        .attr("text-anchor", "middle")
        .attr("x", 15) // 胶囊右侧，更靠右
        .attr("y", 5) // 垂直居中
        .attr("font-size", "14px")
        .attr("fill", "#ffffff")
        .text(feature);
    } else {
      // 添加左侧文本
      label
        .append("text")
        .attr("text-anchor", "middle")
        .attr("x", -15)
        .attr("y", 5)
        .attr("font-size", "14px")
        .attr("fill", "#ffffff")
        .text(feature);

      // 数字的圆形背景
      label
        .append("circle")
        .attr("r", 13)
        .attr("cx", 15)
        .attr("cy", 0)
        .style("fill", bgColor);

      // 添加分数值
      label
        .append("text")
        .attr("text-anchor", "middle")
        .attr("x", 15)
        .attr("y", 5)
        .attr("font-size", "14px")
        .attr("font-weight", "bold")
        .attr("fill", "#ffffff")
        .text(value.toFixed(1));
    }
  });
};

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  if (radarChart) {
    radarChart.dispose();
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

#radar-chart svg {
  position: absolute;
  top: 0;
  left: 0;
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

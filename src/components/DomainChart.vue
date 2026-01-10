<template>
  <div class="mt-6 bg-white rounded-lg shadow p-6">
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import {
  Chart,
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale,
  Legend,
  Title,
  Tooltip,
} from "chart.js";

Chart.register(
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale,
  Legend,
  Title,
  Tooltip
);

interface Dataset {
  label: string;
  data: (number | null)[];
  borderColor: string;
  backgroundColor: string;
  tension: number;
  spanGaps?: boolean;
  baselineRank?: number;
}

const props = defineProps<{
  labels: string[];
  datasets: Dataset[];
}>();

const chartCanvas = ref<HTMLCanvasElement | null>(null);
let chartInstance: Chart | null = null;

const baselinePlugin = {
  id: "baselinePlugin",
  afterDatasetsDraw: (chart: Chart) => {
    const ctx = chart.ctx;
    const yScale = chart.scales.y;
    const xScale = chart.scales.x;
    if (!yScale || !xScale) return;

    const startX = xScale.left;
    const endX = xScale.right;

    chart.data.datasets.forEach((ds) => {
      const baselineRank = (ds as any).baselineRank;
      if (baselineRank === undefined) return;

      const y = yScale.getPixelForValue(baselineRank);

      ctx.save();

      ctx.beginPath();
      ctx.moveTo(startX, y);
      ctx.lineTo(endX, y);
      ctx.strokeStyle = ds.borderColor as string;
      ctx.lineWidth = 2;
      ctx.setLineDash([]);
      ctx.stroke();

      ctx.beginPath();
      ctx.arc(startX, y, 5, 0, Math.PI * 2);
      ctx.fillStyle = ds.borderColor as string;
      ctx.fill();

      ctx.restore();
    });
  },
};

onMounted(() => {
  if (!chartCanvas.value) return;

  chartInstance = new Chart(chartCanvas.value, {
    type: "line",
    data: {
      labels: props.labels,
      datasets: props.datasets.map((ds) => ({ ...ds, spanGaps: true })),
    },
    options: {
      responsive: true,
      plugins: {
        legend: { display: true },
        tooltip: {
          callbacks: {
            label: (context) => {
              const value = context.raw;
              return (
                context.dataset.label +
                ": " +
                (value === null ? "No rank" : value)
              );
            },
          },
        },
      },
      scales: {
        y: {
          min: 1,
          reverse: false,
          title: { display: true, text: "Rank" },
          ticks: {
            callback: (value) => (Number.isInteger(value) ? value : ""),
          },
        },
        x: { title: { display: true, text: "Date" } },
      },
    },
    plugins: [baselinePlugin],
  });
});

watch(
  () => props.labels,
  (newLabels) => {
    if (!chartInstance) return;
    chartInstance.data.labels = newLabels;
    chartInstance.update();
  }
);

watch(
  () => props.datasets,
  (newDatasets) => {
    if (!chartInstance) return;
    chartInstance.data.datasets = newDatasets.map((ds) => ({
      ...ds,
      spanGaps: true,
    }));
    chartInstance.update();
  },
  { deep: true }
);
</script>

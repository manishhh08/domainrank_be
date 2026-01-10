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

const props = defineProps<{
  labels: string[];
  datasets: {
    label: string;
    data: (number | null)[];
    borderColor: string;
    backgroundColor: string;
    tension: number;
  }[];
}>();

const chartCanvas = ref<HTMLCanvasElement | null>(null);
let chartInstance: Chart | null = null;

onMounted(() => {
  if (chartCanvas.value) {
    chartInstance = new Chart(chartCanvas.value, {
      type: "line",
      data: {
        labels: props.labels,
        datasets: props.datasets,
      },
      options: {
        responsive: true,
        plugins: {
          legend: { display: true },
          tooltip: {
            callbacks: {
              label: function (context) {
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
            reverse: false,
            title: { display: true, text: "Rank" },
            ticks: {
              callback: function (value) {
                return Number.isInteger(value) ? value : "";
              },
            },
          },
          x: { title: { display: true, text: "Date" } },
        },
      },
    });
  }
});

// Watch for updates in labels or datasets
watch(
  () => props.labels,
  (newLabels) => {
    if (chartInstance) {
      chartInstance.data.labels = newLabels;
      chartInstance.update();
    }
  }
);

watch(
  () => props.datasets,
  (newDatasets) => {
    if (chartInstance) {
      chartInstance.data.datasets = newDatasets;
      chartInstance.update();
    }
  },
  { deep: true }
);
</script>

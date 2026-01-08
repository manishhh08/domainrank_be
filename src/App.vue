<template>
  <div class="min-h-screen bg-gray-100">
    <!-- Header -->
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto px-4 py-4">
        <h1 class="text-2xl text-center font-bold text-gray-800">
          Domain Rank Checker
        </h1>
      </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-3xl mx-auto px-4 mt-12">
      <h1 class="text-2xl font-bold text-center mb-5">
        Welcome to the domain ranking checker. We show the latest ranking of
        popular domains in the world.
      </h1>

      <div class="bg-white rounded-lg shadow p-6">
        <h2 class="text-lg font-semibold text-gray-700 mb-4">
          Check Domain Ranking
        </h2>

        <!-- Search Input -->
        <div class="flex flex-col sm:flex-row gap-4">
          <input
            v-model="domain"
            type="text"
            placeholder="Enter domain (e.g. amazon.com)"
            class="flex-1 px-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
          />
          <button
            @click="submit"
            class="px-6 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 transition"
          >
            Search
          </button>
        </div>

        <!-- Static Result -->
        <p class="mt-4 text-sm text-gray-700">
          Latest rank for
          <span class="font-semibold text-gray-900">amazon.com</span>:
          <span class="font-bold text-green-600">24</span>
        </p>
      </div>

      <!-- Line Chart -->
      <div class="mt-6 bg-white rounded-lg shadow p-6">
        <canvas ref="chartCanvas"></canvas>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import {
  Chart,
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale,
} from "chart.js";

Chart.register(
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale
);

const domain = ref("");
const chartCanvas = ref(null);
let chartInstance = null;

onMounted(() => {
  chartInstance = new Chart(chartCanvas.value, {
    type: "line",
    data: {
      labels: ["Day 1", "Day 2", "Day 3", "Day 4", "Day 5"],
      datasets: [
        {
          label: "Tranco Rank",
          data: [30, 28, 26, 25, 24],
          borderColor: "#16a34a",
          backgroundColor: "rgba(16, 163, 127, 0.2)",
          tension: 0.4,
        },
      ],
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          display: true,
        },
      },
      scales: {
        y: {
          reverse: true,
          title: {
            display: true,
            text: "Rank",
          },
        },
      },
    },
  });
});

const submit = () => {
  if (!domain.value) {
    alert("Please enter a domain");
    return;
  }

  console.log("Searching for:", domain.value);
};
</script>

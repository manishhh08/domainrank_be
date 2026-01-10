<template>
  <div class="min-h-screen bg-gray-100">
    <Header />

    <main class="max-w-4xl mx-auto px-4 mt-12">
      <DomainSearch
        :loading="loading"
        :results="results"
        @search="handleSearch"
      />

      <DomainChart :labels="chartLabels" :datasets="chartDataSets" />
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import Header from "./components/Header.vue";
import DomainSearch from "./components/DomainSearch.vue";
import DomainChart from "./components/DomainChart.vue";
import { useDomain } from "./services/useDomain";
import { DomainResult } from "./types/domain";

const loading = ref(false);
const results = ref<DomainResult[]>([]);

const chartLabels = ref<string[]>([]);
const chartDataSets = ref<
  {
    label: string;
    data: (number | null)[];
    borderColor: string;
    backgroundColor: string;
    tension: number;
  }[]
>([]);

// Colors for multiple lines
const colors = [
  "#16a34a",
  "#2563eb",
  "#dc2626",
  "#f59e0b",
  "#8b5cf6",
  "#d946ef",
  "#14b8a6",
  "#f43f5e",
  "#eab308",
  "#0ea5e9",
  "#84cc16",
  "#f97316",
  "#c026d3",
  "#22c55e",
  "#1e3a8a",
];

const { fetchMultipleDomain } = useDomain();

const handleSearch = async (domainInput: string) => {
  loading.value = true;
  results.value = [];
  chartLabels.value = [];
  chartDataSets.value = [];

  const domains = domainInput
    .split(",")
    .map((d) => d.trim())
    .filter(Boolean);

  if (!domains.length) {
    loading.value = false;
    return;
  }

  try {
    const fetchedResults: DomainResult[] = await fetchMultipleDomain(domains);

    if (!fetchedResults.length) return;

    results.value = fetchedResults;

    const allDatesSet = new Set<string>();
    fetchedResults.forEach((r) =>
      r.history.forEach((h) => allDatesSet.add(h.date))
    );
    const allDates = Array.from(allDatesSet).sort();
    chartLabels.value = allDates;

    chartDataSets.value = fetchedResults.map((r, idx) => ({
      label: r.domain,
      data: allDates.map((d) => {
        const h = r.history.find((h) => h.date === d);
        return h ? h.rank : null;
      }),
      borderColor: colors[idx % colors.length],
      backgroundColor: colors[idx % colors.length] + "33",
      tension: 0.4,
      baselineRank: r.history[0]?.rank,
    }));
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};
</script>

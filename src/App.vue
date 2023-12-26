<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const weightChartEl = ref(null);
const weightChart = shallowRef(null);
const weightInput = ref(60.0);
const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.data - b.data)
        .map((w) => new Date(w.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.data - b.data)
        .map((w) => w.weight)
        .slice(-7);
      weightChart.value.update();

      return;
    }

    nextTick(() => {
      // console.log(weightChartEl)
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.data - b.data)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              labels: "weight",
              data: ws.sort((a, b) => a.data - b.data).map((w) => w.weight),
              backgroundColor: "rgba(255,105,180,0.2)",
              borderColor: "rgba(255,105, 180)",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
    // console.log(weightChartEl)
  },
  { deep: true }
);
</script>

<template>
  <main style="display: flex; flex-direction:column; align-items: center;">
    <h2>weight Tracker</h2>
    <div class="curren" style="display: flex; align-items:center">
      <h5>{{ currentWeight.weight }} Current weight (Kg)</h5>
    </div>
    <form @submit.prevent="addWeight">
      <div class="input-group mb-3">
        <input
          type="number"
          class="form-control"
          step="0.1"
          v-model="weightInput"
        />
        <input type="submit" class="btn btn-primary" value="Add weight" />
      </div>
    </form>
    <div v-if="weights && weights.length > 0">
      <h4>Last 7 days</h4>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="weight-history">
        <h4>Weight History</h4>
        <ul>
          <li v-for="weight in weights">
            <span> {{ weight.weight }}Kg </span>
            <small>
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped></style>

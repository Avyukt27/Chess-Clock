<template>
  <div
    class="min-h-screen bg-slate-100 text-black dark:bg-slate-900 dark:text-white grid grid-cols-3 grid-rows-[max-content_1fr] justify-center items-center grid-flow-row"
  >
    <h1 class="text-center text-3xl font-bold p-4 col-span-3">
      Welcome to the Chess Clock!
    </h1>

    <label
      class="text-center text-2xl font-bold font-mono p-6 border-gray-500 border-solid border-2 col-start-1 row-start-2 self-end"
      for="set-time-seconds"
      v-if="!started"
    >
      Start Time
    </label>
    <select
      class="text-center text-2xl bg-black text-white font-bold font-mono p-6 my-4 border-gray-500 border-solid border-2 row-start-3 col-start-1 self-start"
      id="set-time-seconds"
      v-model="selectedTimeSeconds"
      v-if="!started"
    >
      <option :value="1800">30 Minutes</option>
      <option :value="900">15 Minutes</option>
      <option :value="600">10 Minutes</option>
      <option :value="180">3 Minutes</option>
      <option :value="60">1 Minute</option>
      <option :value="30">30 Seconds</option>
    </select>

    <!-- Dropdown to change addingInterval -->
    <label
      class="text-center text-2xl font-bold font-mono p-6 border-gray-500 border-solid border-2 col-start-3 row-start-2 self-end"
      for="interval-select"
      v-if="!started"
    >
      Interval
    </label>
    <select
      class="text-center text-2xl bg-black text-white font-bold font-mono p-6 my-4 border-gray-500 border-solid border-2 row-start-3 col-start-3 self-start"
      id="interval-select"
      v-model="addingInterval"
      v-if="!started"
    >
      <option :value="10">10 Seconds</option>
      <option :value="3">3 Seconds</option>
      <option :value="2">2 Seconds</option>
      <option :value="0">0 Seconds</option>
    </select>

    <button
      class="text-center text-2xl font-bold font-mono p-6 mx-96 border-gray-500 border-solid border-2 col-span-3"
      @click="start"
      v-if="!started"
    >
      Start
    </button>

    <button
      :class="[blackTurn ? activeBlack : inactiveBlack]"
      @click="setTurnWhite"
      v-if="started"
    >
      {{ blackTimeReadable }}
    </button>

    <button
      class="text-center text-2xl font-bold font-mono p-6 mx-5 border-gray-500 border-solid border-2"
      @click="stop"
      v-if="started"
    >
      Back
    </button>

    <button
      :class="[whiteTurn ? activeWhite : inactiveWhite]"
      @click="setTurnBlack"
      v-if="started"
    >
      {{ whiteTimeReadable }}
    </button>
  </div>
</template>

<script setup lang="js">
import { ref, onMounted } from "vue";
import { onClickOutside } from "@vueuse/core";

// Reactive state variables
const started = ref(false);
const whiteTurn = ref(true);
const blackTurn = ref(false);

const whiteTimeSeconds = ref(900);
const blackTimeSeconds = ref(900);
const blackTimeReadable = ref("00:00:00");
const whiteTimeReadable = ref("00:00:00");
const addingInterval = ref(10);

const selectedTimeSeconds = ref(900);

const activeWhite = ref("activeWhite");
const activeBlack = ref("activeBlack");
const inactiveWhite = ref("inactiveWhite");
const inactiveBlack = ref("inactiveBlack");

const dropdownOpen = ref(false);
const target = ref(null);

let timerInterval = null;

// Function to set time in readable format
function setTimeReadable() {
  blackTimeReadable.value =
    Math.floor(blackTimeSeconds.value / 3600)
      .toString()
      .padStart(2, "0") +
    ":" +
    (Math.floor(blackTimeSeconds.value / 60) -
      Math.floor(blackTimeSeconds.value / 3600) * 60)
      .toString()
      .padStart(2, "0") +
    ":" +
    Math.floor(blackTimeSeconds.value % 60)
      .toString()
      .padStart(2, "0");
  whiteTimeReadable.value =
    Math.floor(whiteTimeSeconds.value / 3600)
      .toString()
      .padStart(2, "0") +
    ":" +
    (Math.floor(whiteTimeSeconds.value / 60) -
      Math.floor(whiteTimeSeconds.value / 3600) * 60)
      .toString()
      .padStart(2, "0") +
    ":" +
    Math.floor(whiteTimeSeconds.value % 60)
      .toString()
      .padStart(2, "0");
}

// Function to handle the black turn
function setTurnBlack() {
  if (whiteTurn.value) {
    whiteTimeSeconds.value += addingInterval.value;
    whiteTurn.value = false;
    blackTurn.value = true;
  }
}

// Function to handle the white turn
function setTurnWhite() {
  if (blackTurn.value) {
    blackTimeSeconds.value += addingInterval.value;
    whiteTurn.value = true;
    blackTurn.value = false;
  }
}

// Start timer
function start() {
  whiteTimeSeconds.value = selectedTimeSeconds.value;
  blackTimeSeconds.value = selectedTimeSeconds.value
  setTimeReadable();
  started.value = true;
  timerInterval = setInterval(() => {
    if (blackTurn.value) {
      blackTimeSeconds.value--;
      setTimeReadable();
    } else if (whiteTurn.value) {
      whiteTimeSeconds.value--;
      setTimeReadable();
    }
  }, 1000);
}

// Stop timer
function stop() {
  started.value = false;
  clearInterval(timerInterval);
}

// Close dropdown
function closeDropdown() {
  dropdownOpen.value = false;
}

onMounted(() => {
  onClickOutside(target, () => closeDropdown());
});
</script>

<style lang="css" scoped>
.activeWhite {
  @apply text-black bg-white text-center text-3xl font-bold font-mono h-52 m-4 border-gray-500 border-solid border-2 col-span-3 mx-96;
}

.activeBlack {
  @apply text-white bg-black text-center text-3xl font-bold font-mono h-60 m-4 border-gray-500 border-solid border-2 col-span-3 mx-96;
}

.inactiveWhite {
  @apply text-black bg-white bg-opacity-50 text-center text-2xl font-bold font-mono h-60 m-4 border-gray-500 border-solid border-2 col-span-3 mx-96;
}

.inactiveBlack {
  @apply text-white bg-black bg-opacity-50 text-center text-2xl font-bold font-mono h-52 m-4 border-gray-500 border-solid border-2 col-span-3 mx-96;
}

select {
  margin-bottom: 200px;
}
</style>

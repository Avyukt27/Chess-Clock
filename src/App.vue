<template>
  <div
    class="min-h-screen p-4 bg-slate-100 text-black dark:bg-slate-900 dark:text-white grid grid-cols-3 grid-rows-[max-content_1fr] justify-center items-center grid-flow-row"
  >
    <h1 class="text-center text-3xl font-bold p-4 col-span-3" v-show="!started">
      Welcome to the Chess Clock
    </h1>

    <label
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl font-bold font-mono p-4 sm:p-5 md:p-6 lg:p-6 xl:p-8 border-gray-500 border-solid border-2 col-start-1 row-start-2 self-end"
      for="set-time-seconds"
      v-if="!started"
    >
      Start Time
    </label>
    <select
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl bg-black text-white font-bold font-mono p-6 my-4 border-gray-500 border-solid border-2 row-start-3 col-start-1 self-start"
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
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl font-bold font-mono p-4 sm:p-5 md:p-6 lg:p-6 xl:p-8 border-gray-500 border-solid border-2 col-start-3 row-start-2 self-end"
      for="interval-select"
      v-if="!started"
    >
      Interval
    </label>
    <select
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl bg-black text-white font-bold font-mono p-6 my-4 border-gray-500 border-solid border-2 row-start-3 col-start-3 self-start"
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
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl font-bold font-mono p-6 border-gray-500 border-solid border-2 col-span-3"
      @click="start"
      v-if="!started"
    >
      Start
    </button>

    <button
      :class="{
        [blackTurn ? activeBlack : inactiveBlack]: started,
        [blackWin ? win : lose]: over,
      }"
      @click="setTurn"
      v-show="started"
      :disabled="paused"
    >
      {{ blackTimeReadable }}
    </button>

    <button
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl font-bold font-mono p-4 sm:p-5 md:p-6 lg:p-6 xl:p-8 border-gray-500 border-solid border-2"
      @click="stop"
      v-if="started"
    >
      Back
    </button>

    <button
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl font-bold font-mono p-4 sm:p-5 md:p-6 lg:p-6 xl:p-8 border-gray-500 border-solid border-2 col-start-3"
      @click="pause"
      v-if="started && !paused"
    >
      Pause
    </button>

    <button
      class="text-center text-base sm:text-lg md:text-xl lg:text-2xl xl:text-3xl font-bold font-mono p-4 sm:p-5 md:p-6 lg:p-6 xl:p-8 border-gray-500 border-solid border-2 col-start-3"
      @click="resume"
      v-if="started && paused"
    >
      Resume
    </button>

    <button
      :class="{
        [whiteTurn ? activeWhite : inactiveWhite]: started,
        [whiteWin ? win : lose]: over,
      }"
      @click="setTurn"
      v-show="started"
      :disabled="paused"
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
const over = ref(false);
const whiteTurn = ref(true);
const blackTurn = ref(false);

const whiteTimeSeconds = ref(900);
const blackTimeSeconds = ref(900);
const blackTimeReadable = ref("00:00:00");
const whiteTimeReadable = ref("00:00:00");
const addingInterval = ref(10);

const paused = ref(false);
const selectedTimeSeconds = ref(900);

const activeWhite = ref("activeWhite");
const activeBlack = ref("activeBlack");
const inactiveWhite = ref("inactiveWhite");
const inactiveBlack = ref("inactiveBlack");
const win = ref("win");
const lose = ref("lose");

const whiteWin = ref(false);
const blackWin = ref(false);

const dropdownOpen = ref(false);
const target = ref(null);

let timerInterval = null;

// Set the winner at game over
function gameOver() {
  over.value = true;
  clearInterval(timerInterval);
  if (whiteTurn.value) {
    blackWin.value = true;
    whiteWin.value = false;
  } else if (blackTurn.value) {
    whiteWin.value = true;
    blackTurn.value = false;
  }
}

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
function setTurn() {
  if (whiteTurn.value) {
    whiteTimeSeconds.value += addingInterval.value;
    whiteTurn.value = false;
    blackTurn.value = true;
  } else if (blackTurn.value) {
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
  over.value = false;
  paused.value = false;

  whiteTurn.value = true;
  blackTurn.value = false;
  timerInterval = setInterval(() => {
    if (blackTurn.value) {
      blackTimeSeconds.value--;
      setTimeReadable();
      if (blackTimeSeconds.value <= 0) {
        gameOver();
      }
    } else if (whiteTurn.value) {
      whiteTimeSeconds.value--;
      setTimeReadable();
      if (whiteTimeSeconds.value <= 0) {
        gameOver();
      }
    }
  }, 1000);
}

// Stop timer
function stop() {
  started.value = false;
  over.value = true;
  paused.value = false;
  clearInterval(timerInterval);
}

// Pause timer
function pause() {
  paused.value = true;
  clearInterval(timerInterval);
}

// Resume timer
function resume() {
  paused.value = false;
  timerInterval = setInterval(() => {
    if (blackTurn.value) {
      blackTimeSeconds.value--;
      setTimeReadable();
      if (blackTimeSeconds.value <= 0) {
        gameOver();
      }
    } else if (whiteTurn.value) {
      whiteTimeSeconds.value--;
      setTimeReadable();
      if (whiteTimeSeconds.value <= 0) {
        gameOver();
      }
    }
  }, 1000);
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
  @apply text-black bg-white text-center text-3xl font-bold font-mono h-52 mx-2 sm:mx-3 md:mx-4 lg:mx-4 xl:mx-6 border-gray-500 border-solid border-2 col-span-3;
}

.activeBlack {
  @apply text-white bg-black text-center text-3xl font-bold font-mono h-60 mx-2 sm:mx-3 md:mx-4 lg:mx-4 xl:mx-6 border-gray-500 border-solid border-2 col-span-3;
}

.inactiveWhite {
  @apply text-black bg-white bg-opacity-50 text-center text-2xl font-bold font-mono h-60 mx-2 sm:mx-3 md:mx-4 lg:mx-4 xl:mx-6 border-gray-500 border-solid border-2 col-span-3;
}

.inactiveBlack {
  @apply text-white bg-black bg-opacity-50 text-center text-2xl font-bold font-mono h-52 mx-2 sm:mx-3 md:mx-4 lg:mx-4 xl:mx-6 border-gray-500 border-solid border-2 col-span-3;
}

.win {
  @apply text-white bg-green-500 bg-opacity-50 text-center text-2xl font-bold font-mono h-52 mx-2 sm:mx-3 md:mx-4 lg:mx-4 xl:mx-6 border-gray-500 border-solid border-2 col-span-3;
}

.lose {
  @apply text-black bg-red-500 bg-opacity-50 text-center text-2xl font-bold font-mono h-52 mx-2 sm:mx-3 md:mx-4 lg:mx-4 xl:mx-6 border-gray-500 border-solid border-2 col-span-3;
}

select {
  margin-bottom: 200px;
}
</style>

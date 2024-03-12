<template>
  <div class="p-4 w-[320px] bg-blue-100 rounded-lg shadow mx-auto">
    <h2 class="text-xl font-bold mb-4 text-center">運動計時器</h2>
    <div id="timer-display" class="mt-4 text-lg font-semibold mb-6">
      <span class="text-4xl">{{ displayTime }} 秒</span>
      <p v-if="isRunning" class="text-xs mt-2">
        現在是
        <span class="text-blue-600" v-if="currentPhase === 'workout'"
          >運動</span
        >
        <span
          class="text-green-600"
          v-else-if="currentPhase === 'restBetweenSets'"
          >組間休息</span
        >
        <span class="text-red-600" v-else>休息</span> 時間，剩餘
        {{ phaseTimeRemaining }} 秒
      </p>
      <p v-if="isCompleted" class="text-lg text-green-600 text-center my-4">
        恭喜，運動計畫完成！
      </p>
      <p class="text-xs mt-2">已完成運動數量: {{ currentExerciseCount }}</p>
      <p class="text-xs mt-2">已完成運動組數: {{ currentSetCount }}</p>
    </div>
    <div class="mb-3">
      <label for="workout-time" class="block text-sm mb-2"
        >運動時間 (秒):</label
      >
      <input
        v-model.number="workoutTime"
        type="number"
        id="workout-time"
        min="1"
        class="w-full p-2 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
    </div>
    <div class="mb-3">
      <label for="rest-time" class="block text-sm mb-2"
        >運動間休息時間 (秒):</label
      >
      <input
        v-model.number="restTime"
        type="number"
        id="rest-time"
        min="1"
        class="w-full p-2 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
    </div>
    <div class="mb-3">
      <label for="exercises-per-set" class="block text-sm mb-2"
        >一組包含幾項運動:</label
      >
      <input
        v-model.number="exercisesPerSet"
        type="number"
        id="exercises-per-set"
        min="1"
        class="w-full p-2 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
    </div>
    <div class="mb-3">
      <label for="rest-between-sets" class="block text-sm mb-2"
        >組間休息時間 (秒):</label
      >
      <input
        v-model.number="restBetweenSets"
        type="number"
        id="rest-between-sets"
        min="1"
        class="w-full p-2 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
    </div>
    <div class="mb-3">
      <label for="number-of-sets" class="block text-sm mb-2">訓練組數:</label>
      <input
        v-model.number="numberOfSets"
        type="number"
        id="number-of-sets"
        min="1"
        class="w-full p-2 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
    </div>
    <div class="flex justify-between">
      <button
        @click="startTimer"
        :disabled="isRunning"
        class="bg-blue-500 text-white px-3 py-1 text-sm rounded hover:bg-blue-400 disabled:opacity-50"
      >
        啟動
      </button>
      <button
        @click="pauseTimer"
        :disabled="!isRunning"
        class="bg-red-500 text-white px-3 py-1 text-sm rounded hover:bg-red-400 disabled:opacity-50"
      >
        暫停
      </button>
      <button
        @click="resetTimer"
        class="bg-gray-500 text-white px-3 py-1 text-sm rounded hover:bg-gray-400"
        :disabled="isRunning"
      >
        重置
      </button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onUnmounted } from "vue";

export default {
  name: "TimerComponent",
  setup() {
    const workoutTime = ref(30);
    const restTime = ref(10);
    const restBetweenSets = ref(60);
    const exercisesPerSet = ref(4);
    const numberOfSets = ref(3);
    const isRunning = ref(false);
    const currentPhase = ref("workout");
    const phaseTimeRemaining = ref(workoutTime.value);
    const currentExerciseCount = ref(0);
    const currentSetCount = ref(0);
    const isCompleted = ref(false);

    let lastTimestamp = 0;
    let rafId = null;

    const updateTimer = (timestamp) => {
      if (!lastTimestamp) lastTimestamp = timestamp;
      const progress = timestamp - lastTimestamp;

      if (progress >= 1000) {
        lastTimestamp = timestamp;
        phaseTimeRemaining.value -= 1;

        if (phaseTimeRemaining.value <= 0) {
          switchPhase();
        }
      }

      if (isRunning.value) {
        rafId = requestAnimationFrame(updateTimer);
      }
    };

    function startTimer() {
      if (!isRunning.value) {
        isRunning.value = true;
        requestAnimationFrame(updateTimer);
      }
    }

    function pauseTimer() {
      isRunning.value = false;
      if (rafId) {
        cancelAnimationFrame(rafId);
        rafId = null;
      }
    }

    function resetTimer() {
      if (rafId) {
        cancelAnimationFrame(rafId);
        rafId = null;
      }
      isRunning.value = false;
      currentPhase.value = "workout";
      phaseTimeRemaining.value = workoutTime.value; // 重置為運動時間的初始值
      currentExerciseCount.value = 0;
      currentSetCount.value = 0;
    }

    function switchPhase() {
      // 檢查是否完成所有訓練組數
      if (
        currentSetCount.value === numberOfSets.value &&
        currentPhase.value !== "workout"
      ) {
        pauseTimer(); // 自動暫停
        isCompleted.value = true; // 標記運動完成
        return; // 結束函數執行
      }
      if (currentPhase.value === "workout") {
        currentExerciseCount.value += 1;
        if (
          currentExerciseCount.value % exercisesPerSet.value === 0 &&
          currentSetCount.value < numberOfSets.value - 1
        ) {
          currentPhase.value = "restBetweenSets";
          phaseTimeRemaining.value = restBetweenSets.value;
          currentSetCount.value += 1;
        } else if (
          currentExerciseCount.value % exercisesPerSet.value !== 0 ||
          currentSetCount.value === numberOfSets.value - 1
        ) {
          currentPhase.value = "rest";
          phaseTimeRemaining.value = restTime.value;
        }
      } else if (currentPhase.value === "restBetweenSets") {
        currentPhase.value = "workout";
        phaseTimeRemaining.value = workoutTime.value;
      } else {
        currentPhase.value = "workout";
        phaseTimeRemaining.value = workoutTime.value;
      }
    }

    const displayTime = computed(() => {
      return currentPhase.value === "workout"
        ? phaseTimeRemaining.value
        : phaseTimeRemaining.value;
    });

    onUnmounted(() => {
      if (rafId) {
        cancelAnimationFrame(rafId);
      }
    });

    return {
      workoutTime,
      restTime,
      restBetweenSets,
      exercisesPerSet,
      numberOfSets,
      startTimer,
      pauseTimer,
      resetTimer,
      isRunning,
      currentPhase,
      phaseTimeRemaining,
      currentExerciseCount,
      currentSetCount,
      displayTime,
      isCompleted,
    };
  },
};
</script>

<style scoped>
/* Tailwind CSS 已直接應用於模板，無需額外樣式 */
</style>

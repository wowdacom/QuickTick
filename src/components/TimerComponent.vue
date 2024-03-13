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
        {{ workoutTimeRemaining }} 秒
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
        v-model.number="unitWorkoutTime"
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
        v-model.number="unitRestTime"
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
    const unitWorkoutTime = ref(5);
    const unitRestTime = ref(2);
    const exercisesPerSet = ref(3);
    const restBetweenSets = ref(60);
    const numberOfSets = ref(2);
    const isRunning = ref(false);
    const currentPhase = ref("stopped");
    const workoutTimeRemaining = ref(unitWorkoutTime.value);
    const restTimeRemaining = ref(unitRestTime.value);
    const restTimeBetweenSetsRemaining = ref(restBetweenSets.value);
    const currentExerciseCount = ref(0);
    const currentSetCount = ref(0);
    const isCompleted = ref(false);

    let lastSecond = -1; // 初始化為 -1，以確保第一次更新時能夠輸出
    const rafId = ref(null);

    const updateTimer = (timestamp) => {
      let currentTime = Math.floor(timestamp / 1000); // 使用 Math.floor 計算秒

      if (currentTime !== lastSecond) {
        lastSecond = currentTime; // 更新 lastSecond 為當前秒數
        switchPhase(); // 檢查是否需要切換階段`
      }
      rafId.value = window.requestAnimationFrame(updateTimer); // 繼續下一次動畫幀的請求
    };

    function startTimer() {
      if (isRunning.value) return;

      isRunning.value = true;
      currentPhase.value = "workout";
      rafId.value = requestAnimationFrame(updateTimer);
    }

    function pauseTimer() {
      if (!isRunning.value) return;
      isRunning.value = false;
      cancelAnimationFrame(rafId.value);
    }

    function resetTimer() {
      pauseTimer();
      isRunning.value = false;
      currentPhase.value = "stopped";
      workoutTimeRemaining.value = unitWorkoutTime.value;
      restTimeRemaining.value = unitRestTime.value;
      restTimeBetweenSetsRemaining.value = restBetweenSets.value;
      currentExerciseCount.value = 0;
      currentSetCount.value = 0;
      isCompleted.value = false;
    }

    function switchPhase() {
      if (currentPhase.value === "stopped") return;
      if (isCompleted.value) {
        pauseTimer();
        return;
      }

      // 運動中 workout
      if (currentPhase.value === "workout") {
        if (workoutTimeRemaining.value > 0) {
          workoutTimeRemaining.value -= 1;
        } else {
          currentExerciseCount.value += 1;
          if (currentExerciseCount.value % exercisesPerSet.value === 0) {
            currentSetCount.value += 1;
            if (currentSetCount.value === numberOfSets.value) {
              isCompleted.value = true;
              pauseTimer();
            } else {
              currentPhase.value = "restBetweenSets";
              restTimeBetweenSetsRemaining.value = restBetweenSets.value;
            }
          } else {
            currentPhase.value = "rest";
            restTimeRemaining.value = unitRestTime.value;
          }
        }
      }

      // 運動之間休息 rest
      if (currentPhase.value === "rest") {
        if (restTimeRemaining.value > 0) {
          restTimeRemaining.value -= 1;
        } else {
          currentPhase.value = "workout";
          workoutTimeRemaining.value = unitWorkoutTime.value;
        }
      }

      // 組間休息 restBetweenSets
      if (currentPhase.value === "restBetweenSets") {
        if (restTimeBetweenSetsRemaining.value > 0) {
          restTimeBetweenSetsRemaining.value -= 1;
        } else {
          currentPhase.value = "workout";
          workoutTimeRemaining.value = unitWorkoutTime.value;
        }
      }
    }

    const displayTime = computed(() => {
      if (currentPhase.value === "workout") {
        return workoutTimeRemaining.value;
      } else if (currentPhase.value === "rest") {
        return restTimeRemaining.value;
      } else if (currentPhase.value === "restBetweenSets") {
        return restTimeBetweenSetsRemaining.value;
      } else {
        return "--";
      }
    });

    return {
      unitWorkoutTime,
      unitRestTime,
      restBetweenSets,
      exercisesPerSet,
      numberOfSets,
      startTimer,
      pauseTimer,
      resetTimer,
      isRunning,
      currentPhase,
      workoutTimeRemaining,
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

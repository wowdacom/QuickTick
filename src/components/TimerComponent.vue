<template>
  <div class="p-4 w-[320px] bg-blue-100 rounded-lg shadow mx-auto">
    <h2 class="text-xl font-bold mb-4 text-center">運動計時器</h2>
    <div id="timer-display" class="mt-4 text-lg font-semibold mb-6">
      <span class="text-4xl">{{ displayTime }} 秒</span>

      <p class="text-xs mt-2 h-4">
        <template v-if="isRunning">
          現在是
          <span class="text-blue-600" v-if="currentPhase === 'workout'"
            >運動時間，剩餘 {{ workoutTimeRemaining }} 秒</span
          >
          <span
            class="text-green-600"
            v-else-if="currentPhase === 'restBetweenSets'"
            >組間休息時間，剩餘 {{ restTimeBetweenSetsRemaining }} 秒</span
          >
          <span class="text-red-600" v-else-if="currentPhase === 'rest'"
            >休息時間，剩餘 {{ restTimeRemaining }} 秒</span
          >
        </template>
      </p>
      <p v-if="isCompleted" class="text-lg text-green-600 text-center my-4">
        恭喜，運動計畫完成！
      </p>
      <p class="text-xs mt-2">已完成運動數量: {{ currentExerciseCount }}</p>
      <p class="text-xs mt-2">已完成運動組數: {{ currentSetCount }}</p>
      <div class="text-center font-bold text-xl mt-5">運動流程時間軸範例</div>
      <div class="flex justify-center items-center px-10 py-5 relative text-xs">
        <div class="absolute inset-0 flex items-center justify-between">
          <div class="h-1 bg-gray-400 w-full"></div>
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
        >
          第一組開始
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
        >
          運動 <span class="text-[8px]">{{ unitWorkoutTime }}</span>
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
        >
          休息 <span class="text-[8px]">{{ unitRestTime }}</span>
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
        >
          運動 <span class="text-[8px]">{{ unitWorkoutTime }}</span>
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="exercisesPerSet > 2"
        >
          ...
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
        >
          第一組結束
        </div>
        <div
          class="z-10 bg-red-200 border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1"
        >
          組間休息 <span class="text-[8px]">{{ restBetweenSets }}</span>
        </div>

        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1"
        >
          第二組開始
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1"
        >
          運動
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1"
        >
          休息
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1"
        >
          運動
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1 && exercisesPerSet > 2"
        >
          ...
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2"
          v-if="numberOfSets > 1"
        >
          第二組結束
        </div>
        <div
          class="z-10 bg-white border border-gray-300 rounded-lg shadow px-1 py-2 text-base flex flex-col items-center"
          v-if="numberOfSets > 2"
        >
          <Icon icon="material-symbols:restart-alt-rounded" />
          <span class="text-[8px]">>>{{ numberOfSets }}</span>
        </div>
      </div>
    </div>
    <div class="mb-3 flex items-center justify-center">
      <label for="workout-time" class="block text-sm mr-4 w-32 text-right"
        >運動時間</label
      >
      <input
        v-model.number="unitWorkoutTime"
        type="number"
        id="workout-time"
        min="1"
        class="w-16 text-right p-2 mr-4 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
      <div class="w-4">(秒)</div>
    </div>
    <div class="mb-3 flex items-center justify-center">
      <label for="rest-time" class="block text-sm mr-4 w-32 text-right"
        >運動間休息時間</label
      >
      <input
        v-model.number="unitRestTime"
        type="number"
        id="rest-time"
        min="1"
        class="w-16 text-right p-2 mr-4 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
      <div class="w-4">(秒)</div>
    </div>
    <div class="mb-3 flex items-center justify-center">
      <label for="rest-between-sets" class="block text-sm mr-4 w-32 text-right"
        >組間休息時間</label
      >
      <input
        v-model.number="restBetweenSets"
        type="number"
        id="rest-between-sets"
        min="1"
        class="w-16 text-right p-2 mr-4 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
      <div class="w-4">(秒)</div>
    </div>
    <div class="mb-3 flex items-center justify-center">
      <label for="exercises-per-set" class="block text-sm mr-4 w-32 text-right"
        >每組運動總數</label
      >
      <input
        v-model.number="exercisesPerSet"
        type="number"
        id="exercises-per-set"
        min="2"
        class="w-16 text-right p-2 mr-4 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
      <div class="w-4">(項)</div>
    </div>

    <div class="mb-3 flex items-center justify-center">
      <label for="number-of-sets" class="block text-sm mr-4 w-32 text-right"
        >訓練組數</label
      >
      <input
        v-model.number="numberOfSets"
        type="number"
        id="number-of-sets"
        min="1"
        class="w-16 text-right p-2 mr-4 text-sm border rounded disabled:bg-gray-100"
        :disabled="isRunning"
      />
      <div class="w-4">(組)</div>
    </div>
    <div class="flex justify-center">
      <button
        @click="setTimer"
        class="bg-gray-500 text-white px-3 py-1 text-sm rounded hover:bg-gray-400 mr-4"
        :disabled="isRunning"
      >
        設定時間
      </button>
      <button
        @click="pauseTimer"
        :disabled="!isRunning"
        class="bg-red-500 text-white px-3 py-1 text-sm rounded hover:bg-red-400 disabled:opacity-50 mr-4"
      >
        暫停
      </button>
      <button
        @click="startTimer"
        :disabled="isRunning"
        class="bg-blue-500 text-white px-3 py-1 text-sm rounded hover:bg-blue-400 disabled:opacity-50"
      >
        啟動
      </button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onUnmounted } from "vue";
import { Icon } from "@iconify/vue";
import oneSound from "../assets/one.mp3";
import twoSound from "../assets/two.mp3";
import threeSound from "../assets/three.mp3";
import boxingBellSound from "../assets/boxing-bell.mp3";

export default {
  name: "TimerComponent",
  components: {
    Icon,
  },
  setup() {
    const unitWorkoutTime = ref(30);
    const unitRestTime = ref(10);
    const exercisesPerSet = ref(10);
    const restBetweenSets = ref(60);
    const numberOfSets = ref(3);
    const isRunning = ref(false);
    const currentPhase = ref("stopped");
    const workoutTimeRemaining = ref(unitWorkoutTime.value);
    const restTimeRemaining = ref(unitRestTime.value);
    const restTimeBetweenSetsRemaining = ref(restBetweenSets.value);
    const currentExerciseCount = ref(0);
    const currentSetCount = ref(0);
    const isCompleted = ref(false);
    // 初始化 Audio 物件
    const beepSound = ref({
      one: new Audio(oneSound),
      two: new Audio(twoSound),
      three: new Audio(threeSound),
      boxingBellSound: new Audio(boxingBellSound),
    });

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

    const playBeepSound = (name) => {
      pauseTimer();
      beepSound.value[name]
        .play()
        .then(() => {
          isRunning.value = true;
          rafId.value = requestAnimationFrame(updateTimer);
        })
        .catch((error) => console.error("音效播放失敗:", error));
    };

    function pauseTimer() {
      if (!isRunning.value) return;
      isRunning.value = false;
      cancelAnimationFrame(rafId.value);
    }

    function setTimer() {
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
          if (workoutTimeRemaining.value - 1 === 3) {
            playBeepSound("three");
          } else if (workoutTimeRemaining.value - 1 === 2) {
            playBeepSound("two");
          } else if (workoutTimeRemaining.value - 1 === 1) {
            playBeepSound("one");
          } else if (workoutTimeRemaining.value - 1 === 0) {
            playBeepSound("boxingBellSound");
          }
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
          if (restTimeRemaining.value - 1 === 3) {
            playBeepSound("three");
          } else if (restTimeRemaining.value - 1 === 2) {
            playBeepSound("two");
          } else if (restTimeRemaining.value - 1 === 1) {
            playBeepSound("one");
          } else if (restTimeRemaining.value - 1 === 0) {
            playBeepSound("boxingBellSound");
          }
          restTimeRemaining.value -= 1;
        } else {
          currentPhase.value = "workout";
          workoutTimeRemaining.value = unitWorkoutTime.value;
        }
      }

      // 組間休息 restBetweenSets
      if (currentPhase.value === "restBetweenSets") {
        if (restTimeBetweenSetsRemaining.value > 0) {
          if (restTimeBetweenSetsRemaining.value - 1 === 3) {
            playBeepSound("three");
          } else if (restTimeBetweenSetsRemaining.value - 1 === 2) {
            playBeepSound("two");
          } else if (restTimeBetweenSetsRemaining.value - 1 === 1) {
            playBeepSound("one");
          } else if (restTimeBetweenSetsRemaining.value - 1 === 0) {
            playBeepSound("boxingBellSound");
          }
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
      setTimer,
      isRunning,
      currentPhase,
      workoutTimeRemaining,
      currentExerciseCount,
      currentSetCount,
      displayTime,
      isCompleted,
      restTimeRemaining,
      restTimeBetweenSetsRemaining,
      playBeepSound,
    };
  },
};
</script>

<style scoped>
/* Tailwind CSS 已直接應用於模板，無需額外樣式 */
</style>

<script setup lang="ts">
import { computed, ref } from "vue";
import SpinningWheel from "@/components/SpinningWheel.vue";
import WheelOptions from "@/components/WheelOptions.vue";
import WheelControlPanel from "@/components/WheelControlPanel.vue";
import WinnerCard from "@/components/WinnerCard.vue";

const queryWord = ref<string[]>([]);

const wheelOptions = ref<string[]>([
  "Seçenek 1",
  "Seçenek 2",
  "Seçenek 3",
  "Seçenek 4",
  "Seçenek 5",
  "Seçenek 6",
]);

const weights = ref<number[]>([]);
const lucky = ref(true);

const showWinner = ref(false);
const selectedWinner = ref<string | null>(null);

function onSpinEnd(selected: string) {
  console.log("Çark sonucu:", selected);

  selectedWinner.value = selected;
  showWinner.value = true;
}

const wheelSize = computed(() => {
  if (window.innerWidth < 480) return 360;
  if (window.innerWidth < 768) return 430;
  return 500;
});
</script>

<template>
  <div class="flex flex-col lg:flex-row justify-between gap-4 pt-5 w-full max-w-7xl mx-auto px-3">
    <div class="bg-gray-100 p-4 rounded-xl shadow w-full lg:max-w-96">
      <WheelOptions v-model:options="wheelOptions" />
    </div>

    <div class="flex justify-center items-start w-full">
      <SpinningWheel
        :options="wheelOptions"
        :weights="weights"
        :source="queryWord"
        :lucky="lucky"
        :size="wheelSize"
        @spin-end="onSpinEnd"
      />
    </div>
  </div>

  <div class="mt-4 w-full px-3 max-w-5xl mx-auto">
    <WheelControlPanel
      v-model:weights="weights"
      v-model:source="queryWord"
      v-model:lucky="lucky"
      :options="wheelOptions"
    />
  </div>

  <WinnerCard :show="showWinner" :winner="selectedWinner" @close="showWinner = false" />
</template>

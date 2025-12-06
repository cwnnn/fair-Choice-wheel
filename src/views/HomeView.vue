<script setup lang="ts">
import { ref } from "vue";
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
</script>

<template>
  <div class="row pt-5">
    <div class="bg-gray-100 p-4 rounded-xl shadow w-full max-w-96">
      <WheelOptions v-model:options="wheelOptions" />
    </div>

    <div class="flex justify-center items-start flex-1">
      <SpinningWheel
        :options="wheelOptions"
        :weights="weights"
        :source="queryWord"
        :lucky="lucky"
        @spin-end="onSpinEnd"
      />
    </div>
  </div>
  <WheelControlPanel
    v-model:weights="weights"
    v-model:source="queryWord"
    v-model:lucky="lucky"
    :options="wheelOptions"
  />
  <WinnerCard :show="showWinner" :winner="selectedWinner" @close="showWinner = false" />
</template>

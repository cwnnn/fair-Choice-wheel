<script setup lang="ts">
import { ref } from "vue";
import SpinningWheel from "@/components/SpinningWheel.vue";
import WheelOptions from "@/components/WheelOptions.vue";

const wheelOptions = ref<string[]>([
  "Seçenek 1",
  "Seçenek 2",
  "Seçenek 3",
  "Seçenek 4",
  "Seçenek 5",
  "Seçenek 6",
]);

function onSpinEnd(selected: string) {
  console.log("Çark sonucu:", selected);
}
</script>

<template>
  <div class="row pt-5">
    <div class="bg-gray-100 p-4 rounded-xl shadow w-full max-w-96">
      <WheelOptions v-model:options="wheelOptions" />
    </div>

    <div class="flex justify-center items-start flex-1">
      <SpinningWheel :options="wheelOptions" @spin-end="onSpinEnd" />
    </div>
  </div>
</template>

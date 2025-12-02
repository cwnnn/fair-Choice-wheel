<script setup lang="ts">
import SpinningWheel from "@/components/SpinningWheel.vue";
import WheelOptions from "@/components/WheelOptions.vue";
</script>

<template>
  <div class="row pt-5">
    <div class="bg-gray-100 p-4 rounded-xl shadow w-full max-w-96">
      <WheelOptions />
    </div>
    <SpinningWheel
      :options="['Seçenek 1', 'Seçenek 2', 'Seçenek 3', 'Seçenek 4', 'Seçenek 5', 'Seçenek 6']"
    />
  </div>
</template>

<template>
  <!-- Aç/Kapa butonu -->
  <button
    @click="isOpen = true"
    class="fixed bottom-6 right-6 bg-gray-400 text-white w-14 h-14 rounded-full shadow-lg hover:bg-purple-700 flex items-center justify-center text-3xl"
  >
    😈
  </button>

  <!-- Modal -->
  <div
    v-if="isOpen"
    class="fixed inset-0 bg-black/40 backdrop-blur-sm flex items-center justify-center z-50"
  >
    <div
      class="bg-white w-full max-w-lg p-6 rounded-xl shadow-xl relative overflow-y-auto"
      style="max-height: 80vh"
    >
      <!-- Kapat -->
      <button class="absolute right-4 top-4 text-xl" @click="isOpen = false">✖</button>

      <h2 class="text-xl font-bold mb-4">Çark Ayarları</h2>
      <!-- MODE BUTTON GROUP -->
      <n-space class="mb-6">
        <n-button-group>
          <n-button
            ghost
            round
            :type="mode === 'weights' ? 'primary' : 'default'"
            @click="mode = 'weights'"
          >
            Weight Mode
          </n-button>

          <n-button
            ghost
            round
            :type="mode === 'source' ? 'primary' : 'default'"
            @click="mode = 'source'"
          >
            Source Mode
          </n-button>
        </n-button-group>
      </n-space>

      <div v-if="mode === 'weights'" class="space-y-4">
        <div v-for="(opt, i) in options" :key="i">
          <label class="text-black font-medium text-lg">{{ opt }}</label>

          <n-space vertical class="pb-4">
            <span class="text-sm text-gray-600"> Weight ({{ localWeights[i] }}) </span>

            <n-slider v-model:value="localWeights[i]" :min="0" :max="100" :step="1" />

            <n-input-number
              v-model:value="localWeights[i]"
              size="small"
              :min="0"
              :max="100"
              class="w-full"
            />
          </n-space>
        </div>
      </div>

      <!-- SOURCE MODE -->
      <div v-else class="space-y-4 py-4">
        <div class="flex flex-1 justify-between p-4">
          <label class="font-medium">Kelime / Arama Inputu</label>
          <label class="flex items-center gap-2 mt-2">
            <n-switch v-model:value="localLucky" />
            Lucky Aktif
          </label>
        </div>

        <n-input v-model:value="localSource" type="text" placeholder="örn: elma, armut, seçim…" />
      </div>

      <button
        @click="applyChanges"
        class="mt-6 w-full bg-green-600 hover:bg-green-700 text-white font-bold py-2 rounded-lg"
      >
        Uygula
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, defineProps, defineEmits } from "vue";
import { NButton, NButtonGroup, NSpace, NSlider, NInputNumber, NInput, NSwitch } from "naive-ui";

const props = defineProps<{
  options: string[];
  weights: number[];
  source: string[];
  lucky: boolean;
}>();

const emit = defineEmits<{
  (e: "update:weights", v: number[]): void;
  (e: "update:source", v: string[]): void;
  (e: "update:lucky", v: boolean): void;
}>();

const isOpen = ref(false);

const mode = ref<"weights" | "source">("weights");

const localWeights = ref<number[]>([...props.weights]);
const localSource = ref(props.source.join(", "));
const localLucky = ref(props.lucky);

watch(
  () => props.weights,
  (v) => (localWeights.value = [...v])
);

watch(
  () => props.source,
  (v) => (localSource.value = v.join(", "))
);

watch(
  () => props.lucky,
  (v) => (localLucky.value = v)
);
watch(
  () => props.options,
  (opts) => {
    const base = Math.round(100 / opts.length);
    localWeights.value = opts.map(() => base);
  },
  { immediate: true }
);

function applyChanges() {
  if (mode.value === "weights") {
    const base = Math.round(100 / props.options.length);

    const fixed = props.options.map((_, i) => {
      const v = localWeights.value[i];
      return typeof v === "number" ? v : base;
    });

    emit("update:weights", fixed);
    emit("update:source", []);
    emit("update:lucky", false);
  } else {
    const splitWords = localSource.value
      .split(",")
      .map((w) => w.trim())
      .filter((w) => w.length > 0);

    emit("update:source", splitWords);
    emit("update:lucky", localLucky.value);

    emit(
      "update:weights",
      props.options.map(() => 100)
    );
  }

  isOpen.value = false;
}
</script>

<style scoped></style>

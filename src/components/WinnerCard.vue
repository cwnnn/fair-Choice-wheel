<template>
  <div
    v-if="show"
    class="fixed inset-0 bg-black/40 backdrop-blur-sm flex items-center justify-center z-50"
    @click.self="$emit('close')"
  >
    <div class="bg-white rounded-2xl p-6 shadow-2xl w-[320px] text-center relative animate-scale">
      <!-- Sağ üst X -->
      <button
        @click="$emit('close')"
        class="absolute top-3 right-3 text-gray-400 hover:text-gray-600 text-xl"
      >
        ✖
      </button>

      <h2 class="text-2xl font-bold text-gray-800 mb-3">Kazanan!</h2>

      <div class="text-3xl font-extrabold text-green-600 p-6 mb-6">
        {{ winner }}
      </div>

      <button
        @click="$emit('close')"
        class="px-5 py-2 bg-blue-600 text-white rounded-xl shadow hover:bg-blue-700 transition"
      >
        Tamam
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import confetti from "canvas-confetti";
import { watch } from "vue";

const props = defineProps<{
  show: boolean;
  winner: string | null;
}>();

defineEmits(["close"]);

watch(
  () => props.show,
  (isOpen) => {
    if (isOpen) {
      setTimeout(() => fireCelebration(), 150);
    }
  }
);

function fireCelebration() {
  const count = 200;
  const defaults = {
    origin: { y: 0.7 },
  };

  function fire(particleRatio: number, opts: Record<string, unknown>) {
    confetti({
      ...defaults,
      ...opts,
      particleCount: Math.floor(count * particleRatio),
    });
  }

  fire(0.25, {
    spread: 26,
    startVelocity: 55,
  });

  fire(0.2, {
    spread: 60,
  });

  fire(0.35, {
    spread: 100,
    decay: 0.91,
    scalar: 0.8,
  });

  fire(0.1, {
    spread: 120,
    startVelocity: 25,
    decay: 0.92,
    scalar: 1.2,
  });

  fire(0.1, {
    spread: 120,
    startVelocity: 45,
  });
}
</script>

<style scoped>
@keyframes scaleIn {
  0% {
    transform: scale(0.6);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
.animate-scale {
  animation: scaleIn 0.25s ease-out;
}
</style>

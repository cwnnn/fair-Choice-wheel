<template>
  <div :class="['relative flex flex-col items-center justify-center', sizeClass]">
    <canvas ref="canvasRef" :width="size" :height="size" class="rounded-full bg-white"></canvas>
    <img src="./icons/images.png" alt="" class="absolute inset-10 m-auto w-12 h-16 rotate-320" />
    <button
      class="absolute z-10 left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 inset-0 m-auto w-16 h-16 text-blue-900 rounded-full flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
      @click="spin"
      :disabled="isSpinning"
    >
      <span v-if="!isSpinning"
        ><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" class="w-10 h-10">
          <path
            fill="currentColor"
            d="M512 256c0 141.4-114.6 256-256 256S0 397.4 0 256S114.6 0 256 0S512 114.6 512 256zM188.3 147.1c-7.6 4.2-12.3 12.3-12.3 20.9V344c0 8.7 4.7 16.7 12.3 20.9s16.8 4.1 24.3-.5l144-88c7.1-4.4 11.5-12.1 11.5-20.5s-4.4-16.1-11.5-20.5l-144-88c-7.4-4.5-16.7-4.7-24.3-.5z"
          />
        </svg>
      </span>
      <span v-else
        ><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" class="w-8 h-8">
          <path
            fill="currentColor"
            d="M512 256c0 141.4-114.6 256-256 256S0 397.4 0 256S114.6 0 256 0S512 114.6 512 256zM188.3 147.1c-7.6 4.2-12.3 12.3-12.3 20.9V344c0 8.7 4.7 16.7 12.3 20.9s16.8 4.1 24.3-.5l144-88c7.1-4.4 11.5-12.1 11.5-20.5s-4.4-16.1-11.5-20.5l-144-88c-7.4-4.5-16.7-4.7-24.3-.5z"
          />
        </svg>
      </span>
    </button>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch, onMounted, defineProps, defineEmits, computed } from "vue";

const props = defineProps<{
  options: string[];
  size?: number; // default 500
}>();
const emit = defineEmits<{
  (e: "spin-end", selected: string): void;
}>();

const size = props.size ?? 500;
const sizeClass = computed(() => `w-[${size}px] h-[${size}px]`);
const canvasRef = ref<HTMLCanvasElement | null>(null);

const baseColors = [
  "#d60056", // Pembemsi kırmızı (canlı)
  "#ed4f24", // Turuncu (canlı)
  "#fcc603", // Sarı (parlak)
  "#61c42f", // Yeşil (canlı)
  "#5682e8", // Açık mavi (canlı)
  "#5e0afa", // Mavimsi mor (canlı)
];

function generateColors(count: number): string[] {
  if (count <= baseColors.length) return baseColors.slice(0, count);

  const hslColors = baseColors.map(hexToHSL);
  const result: string[] = [];

  for (let i = 0; i < count; i++) {
    // position from 0 to (baseColors.length - 1)
    const t = (i / (count - 1)) * (hslColors.length - 1);
    const idx = Math.floor(t);
    const frac = t - idx;

    const c1 = hslColors[idx];
    const c2 = hslColors[Math.min(idx + 1, hslColors.length - 1)];

    // Hue'yi 0-360 wrap-around ile düzgün interpolate et
    let dh = c2!.h - c1!.h;
    if (dh > 180) dh -= 360;
    else if (dh < -180) dh += 360;

    result.push(
      hslToHex({
        h: (c1!.h + dh * frac + 360) % 360,
        s: c1!.s + (c2!.s - c1!.s) * frac,
        l: c1!.l + (c2!.l - c1!.l) * frac,
      })
    );
  }

  return result;
}

// Örnek hslToHex helper
function hslToHex({ h, s, l }: { h: number; s: number; l: number }) {
  s /= 100;
  l /= 100;
  const c = (1 - Math.abs(2 * l - 1)) * s;
  const x = c * (1 - Math.abs(((h / 60) % 2) - 1));
  const m = l - c / 2;
  let r = 0,
    g = 0,
    b = 0;

  if (h < 60) [r, g, b] = [c, x, 0];
  else if (h < 120) [r, g, b] = [x, c, 0];
  else if (h < 180) [r, g, b] = [0, c, x];
  else if (h < 240) [r, g, b] = [0, x, c];
  else if (h < 300) [r, g, b] = [x, 0, c];
  else [r, g, b] = [c, 0, x];

  const toHex = (v: number) => {
    const hex = Math.round((v + m) * 255).toString(16);
    return hex.length === 1 ? "0" + hex : hex;
  };

  return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
}

// Helper: Convert hex to HSL
function hexToHSL(hex: string) {
  let r = 0,
    g = 0,
    b = 0;
  if (hex.length === 7) {
    r = parseInt(hex.slice(1, 3), 16) / 255;
    g = parseInt(hex.slice(3, 5), 16) / 255;
    b = parseInt(hex.slice(5, 7), 16) / 255;
  }
  const max = Math.max(r, g, b),
    min = Math.min(r, g, b);
  let h = 0,
    s = 0;
  const l = (max + min) / 2;
  if (max !== min) {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch (max) {
      case r:
        h = (g - b) / d + (g < b ? 6 : 0);
        break;
      case g:
        h = (b - r) / d + 2;
        break;
      case b:
        h = (r - g) / d + 4;
        break;
    }
    h /= 6;
  }
  return { h: h * 360, s: s * 100, l: l * 100 };
}

const colors = computed(() => generateColors(props.options.length));

// Animation state
const isSpinning = ref(false);
const rotation = ref(0); // Current rotation angle in radians
const spinVelocity = ref(0); // Current angular velocity
const targetAngle = ref(0); // Where to stop
const selectedIdx = ref<number | null>(null);

function drawWheel() {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  if (!ctx) return;
  ctx.clearRect(0, 0, size, size);
  const center = size / 2;
  const radius = center;
  const n = props.options.length;
  const anglePerSlice = (2 * Math.PI) / n;
  // Draw slices
  const gap = 0.004; // rad cinsinden beyaz boşluk
  for (let i = 0; i < n; i++) {
    ctx.save();
    ctx.beginPath();
    ctx.moveTo(center, center);
    // Dilim açısını gap kadar küçült
    ctx.arc(
      center,
      center,
      radius,
      rotation.value + i * anglePerSlice + gap / 2,
      rotation.value + (i + 1) * anglePerSlice - gap / 2
    );
    ctx.closePath();
    ctx.fillStyle = colors.value[i]!;
    ctx.fill();
    ctx.restore();

    // Draw text
    ctx.save();
    ctx.translate(center, center);
    ctx.rotate(rotation.value + (i + 0.5) * anglePerSlice);
    ctx.textAlign = "right";
    ctx.font = `bold ${Math.floor(size / 18)}px sans-serif`;
    ctx.fillStyle = "#ffffff";
    ctx.fillText(props.options[i]!, radius - 20, 10);
    ctx.restore();
  }

  // Draw center circle
  ctx.save();
  ctx.beginPath();
  ctx.arc(center, center, 40, 0, 2 * Math.PI);
  ctx.fillStyle = "#fff";

  ctx.shadowBlur = 8;
  ctx.fill();
  ctx.restore();
}
onMounted(() => {
  drawWheel();
  animate(); // Idle animasyon sürekli çalışacak
});

const idleVelocity = 0.004; // Çok yavaş dönen hız

// Animation loop
let animFrame: number | null = null;
function animate() {
  if (isSpinning.value) {
    // Spin sırasında normal hız ve frenleme
    rotation.value += spinVelocity.value;
    spinVelocity.value *= 0.985; // Friction
    const diff =
      (((rotation.value - targetAngle.value) % (2 * Math.PI)) + 2 * Math.PI) % (2 * Math.PI);
    if (spinVelocity.value < 0.01 && diff < 0.05) {
      rotation.value = targetAngle.value;
      isSpinning.value = false;
      const n = props.options.length;
      const anglePerSlice = (2 * Math.PI) / n;
      const idx = (n - (Math.floor(rotation.value / anglePerSlice) % n)) % n;
      selectedIdx.value = idx;
      emit("spin-end", props.options[idx]!);
    }
  } else {
    // Idle rotation: yavaş dön
    rotation.value += idleVelocity;
  }

  drawWheel();
  animFrame = requestAnimationFrame(animate);
}

function spin() {
  if (isSpinning.value) return;
  isSpinning.value = true;
  selectedIdx.value = null;
  // Randomly pick a target slice
  const n = props.options.length;
  const anglePerSlice = (2 * Math.PI) / n;
  const targetIdx = Math.floor(Math.random() * n);
  // The pointer is at angle 0, so target angle is:
  // (n - targetIdx) * anglePerSlice + random offset within slice
  const offset = Math.random() * anglePerSlice;
  targetAngle.value = ((n - targetIdx) * anglePerSlice + offset) % (2 * Math.PI);
  // Start with a few spins
  const spins = 5 + Math.random() * 2;
  rotation.value = rotation.value % (2 * Math.PI);
  targetAngle.value += 2 * Math.PI * spins;
  spinVelocity.value = 0.35 + Math.random() * 0.1;
  if (animFrame !== null) {
    cancelAnimationFrame(animFrame);
    animFrame = null; // optional, temizlemek için
  }

  animFrame = requestAnimationFrame(animate);
}

onMounted(drawWheel);
watch(() => props.options, drawWheel, { deep: true });
watch(rotation, drawWheel);
</script>

<style scoped></style>

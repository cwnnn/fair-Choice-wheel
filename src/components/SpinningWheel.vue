<template>
  <div
    :style="{ width: size + 'px', height: size + 'px' }"
    class="relative flex flex-col items-center justify-center"
  >
    <!-- canvas -->
    <canvas
      ref="canvasRef"
      :width="size"
      :height="size"
      class="rounded-full bg-gray-300"
      :style="{ transform: `rotate(${rotationDeg}deg)` }"
    ></canvas>

    <!-- pointer(sadece görsel) -->
    <img
      ref="pointerRef"
      src="./icons/images.png"
      alt=""
      class="absolute inset-5 inset-y-5 lg:inset-13 lg:inset-y-10 m-auto w-12 h-16"
      :style="{ transform: `rotate(${pointerDeg}deg)` }"
    />

    <!-- Spin butonu -->
    <button
      class="absolute z-10 left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 inset-0 m-auto w-16 h-16 text-blue-900 rounded-full flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
      @click="spin()"
      :disabled="isSpinning || pointerDeg !== 315"
    >
      <span v-if="!isSpinning && pointerDeg === 315"
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
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import Fuse from "fuse.js";

const props = withDefaults(
  defineProps<{
    options: string[];
    weights?: number[];
    size?: number;
    source?: string[];
    lucky?: boolean;
  }>(),
  {
    lucky: true,
  }
);

const emit = defineEmits<{
  (e: "spin-end", selected: string): void;
}>();

const size = props.size ?? 500;
const canvasRef = ref<HTMLCanvasElement | null>(null);

const baseColors = ["#d60056", "#ed4f24", "#fcc603", "#61c42f", "#5682e8", "#5e0afa"];

const pointerRef = ref<HTMLImageElement | null>(null); // Gösterge oku için ref
const pointerDeg = ref(315); // başlangıç açısı

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

function generateColors(count: number) {
  const base = baseColors.slice();
  if (count <= base.length) return base.slice(0, count);
  const hsl = base.map(hexToHSL);
  const res: string[] = [];
  for (let i = 0; i < count; i++) {
    const t = (i / (count - 1)) * (hsl.length - 1);
    const idx = Math.floor(t);
    const frac = t - idx;
    const c1 = hsl[idx];
    const c2 = hsl[Math.min(idx + 1, hsl.length - 1)];
    let dh = c2!.h - c1!.h;
    if (dh > 180) dh -= 360;
    else if (dh < -180) dh += 360;
    res.push(
      hslToHex({
        h: (c1!.h + dh * frac + 360) % 360,
        s: c1!.s + (c2!.s - c1!.s) * frac,
        l: c1!.l + (c2!.l - c1!.l) * frac,
      })
    );
  }
  return res;
}

const colors = computed(() => generateColors(props.options.length));

// ---------- canvas çizimi (statik) ----------
function drawWheel() {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  if (!ctx) return;

  const w = canvas.width;
  const h = canvas.height;
  ctx.clearRect(0, 0, w, h);

  const center = w / 2;
  const radius = center;
  const n = props.options.length;
  if (n === 0) return;

  const anglePerSlice = (2 * Math.PI) / n;
  const startAngle = -Math.PI / 2 - anglePerSlice / 2;
  const gap = 0.004;

  for (let i = 0; i < n; i++) {
    const sliceStart = startAngle + i * anglePerSlice + gap / 2;
    const sliceEnd = startAngle + (i + 1) * anglePerSlice - gap / 2;

    // slice
    ctx.save();
    ctx.beginPath();
    ctx.moveTo(center, center);
    ctx.arc(center, center, radius, sliceStart, sliceEnd);
    ctx.closePath();
    ctx.fillStyle = colors.value[i]!;
    ctx.fill();
    ctx.restore();

    // text
    ctx.save();
    ctx.translate(center, center);
    const textAngle = startAngle + (i + 0.5) * anglePerSlice;
    ctx.rotate(textAngle);
    ctx.textAlign = "right";

    const minFontSize = 10,
      maxFontSize = 24,
      idealSlices = 20;
    const fontSize = Math.floor(
      Math.max(minFontSize, Math.min(maxFontSize, maxFontSize * (idealSlices / n)))
    );

    ctx.font = `bold ${fontSize}px sans-serif`;
    ctx.fillStyle = "#ffffff";
    ctx.fillText(props.options[i]!, radius - 20, 10);
    ctx.restore();
  }

  // center circle
  ctx.save();
  ctx.beginPath();
  ctx.arc(center, center, 40, 0, Math.PI * 2);
  ctx.fillStyle = "#fff";
  ctx.shadowBlur = 8;
  ctx.fill();
  ctx.restore();
}

// ---------- dönüş state / animasyon ----------
const rotationDeg = ref(0);
const isSpinning = ref(false);

const animState = {
  startDeg: 0,
  endDeg: 0,
  startTime: 0,
  duration: 0,
};

let rafId: number | null = null;

function easeOutCubic(t: number) {
  return 1 - Math.pow(1 - t, 3);
}

function normalizeDeg(deg: number) {
  let d = deg % 360;
  if (d < 0) d += 360;
  return d;
}

function animationLoop() {
  const now = performance.now();

  if (isSpinning.value) {
    const elapsed = now - animState.startTime;
    const progress = Math.min(1, elapsed / animState.duration);
    const eased = easeOutCubic(progress);

    rotationDeg.value = animState.startDeg + (animState.endDeg - animState.startDeg) * eased;

    if (progress >= 1) {
      isSpinning.value = false;
      rotationDeg.value = normalizeDeg(rotationDeg.value);

      const n = props.options.length;
      if (n > 0) {
        const anglePerSliceDeg = 360 / n;
        const rawIndex = -(rotationDeg.value + 45) / anglePerSliceDeg;
        const rounded = Math.round(rawIndex);
        const idx = ((rounded % n) + n) % n;
        const selected = props.options[idx]!;
        emit("spin-end", selected);
      }
    }
  } else {
    // idle slow rotation
    if (!isSpinning.value) {
      rotationDeg.value = normalizeDeg(rotationDeg.value + 0.3);
    }
  }

  rafId = requestAnimationFrame(animationLoop);
}
const autoWeights = computed(() => {
  if (!props.source || props.source.length === 0) {
    return null;
  }

  const fuse = new Fuse(props.options, {
    threshold: 0.4,
    includeScore: true,
  });
  const weights: number[] = [];

  const data = ref<Record<string, number>>({});

  for (let index = 0; index < props.source.length; index++) {
    const element = props.source[index]!;
    const search = fuse.search(element);

    if (search.length > 0) {
      let raw = search[0]!.score;

      if (raw! < 1e-10) raw = 0;

      const percent = Math.round((1 - raw!) * 100);

      data.value[search[0]!.item] = percent!;
    }
  }

  for (let i = 0; i < props.options.length; i++) {
    const opt = props.options[i]!;
    const val = data.value[opt] ?? null;

    if (props.lucky) {
      weights[i] = val ?? 0;
    } else {
      weights[i] = val !== null ? 100 - val : 100;
    }
  }

  return weights;
});

const normalizedWeights = computed(() => {
  const n = props.options.length;

  if (autoWeights.value) {
    const total = autoWeights.value.reduce((a, b) => a + b, 0);
    return autoWeights.value.map((w) => (total === 0 ? 1 / n : w / total));
  }

  // hiç weight gelmemişse eşit dağıt
  if (!props.weights || props.weights.length !== n) {
    return Array(n).fill(1 / n);
  }

  const total = props.weights.reduce((a, b) => a + b, 0);
  return props.weights.map((w) => w / total);
});

function pickIndexByWeight() {
  const weights = normalizedWeights.value;

  const r = Math.random();
  let sum = 0;

  for (let i = 0; i < weights.length; i++) {
    sum += weights[i];
    if (r <= sum) return i;
  }

  return weights.length - 1;
}

function spin() {
  if (isSpinning.value || props.options.length === 0) return;

  // **Şaka koşulu kontrolü**
  const weightsArray: number[] =
    props.weights && props.weights.length > 0
      ? props.weights
      : autoWeights.value && autoWeights.value.length > 0
      ? autoWeights.value
      : [];

  const allZero = weightsArray.length > 0 && weightsArray.every((w) => w === 0);

  const noMatchFromSource = props.source?.length && autoWeights.value?.every((w) => w === 0);

  if (allZero || noMatchFromSource) {
    triggerJokeMode();
    return;
  }
  isSpinning.value = true;

  const n = props.options.length;
  const anglePerSliceDeg = 360 / n;

  const targetIndex = pickIndexByWeight();

  const maxOffset = anglePerSliceDeg / 2 - 1;
  const wobble = (Math.random() * 2 - 1) * maxOffset;
  const targetDeg = -(targetIndex * anglePerSliceDeg) - 45 + wobble;

  const extraSpins = 4 + Math.floor(Math.random() * 3);
  const totalRotation = extraSpins * 360 + targetDeg;

  animState.startDeg = rotationDeg.value;
  animState.endDeg = totalRotation;
  animState.startTime = performance.now();
  animState.duration = 4200;
}

function triggerJokeMode() {
  // Tek opsiyon ve %0 şans -> Şaka moduna gir
  const jokeModes: Array<"flip" | "endless" | "uzunAdam"> = ["flip", "endless", "uzunAdam"];
  const chosenJoke = jokeModes[Math.floor(Math.random() * jokeModes.length)];

  console.log(chosenJoke);

  if (chosenJoke === "endless") {
    // Şaka 1: Sonsuz Döndürme
    isSpinning.value = true;
    const extraSpins = 1000;
    const targetIndex = 0; // Tek opsiyon var aslında (burada önemli değil, dönmeye devam edecek)
    const anglePerSliceDeg = 360 / props.options.length; // 360 derece
    const maxOffset = anglePerSliceDeg / 2 - 1;
    const wobble = (Math.random() * 2 - 1) * maxOffset;
    const targetDeg = -(targetIndex * anglePerSliceDeg) - 45 + wobble;
    const totalRotation = extraSpins * 360 + targetDeg;
    animState.startDeg = rotationDeg.value;
    animState.endDeg = animState.startDeg + totalRotation;
    animState.startTime = performance.now();
    // Animasyonu çok uzun tut (örn: ~17 dakika)
    animState.duration = 4200 * (extraSpins / 6);
    // *Not:* Bu animasyon çok uzun sürdüğü için spin-end emit edilmeden kullanıcı muhtemelen pes edecektir.
  } else if (chosenJoke === "flip") {
    const startDeg = pointerDeg.value;
    const endDeg = 135; // 315 - 180

    const spins = 3 + Math.floor(Math.random() * 2); // 3–4 tur
    const totalRotation = spins * 360 + (endDeg - (startDeg % 360));

    if (pointerRef.value) {
      pointerRef.value.style.transition = "transform 1.3s cubic-bezier(.17,.67,.38,1)";
    }

    pointerDeg.value = startDeg + totalRotation;

    const sillyWords = ["Patates", "Salatalık", "Bardak"];
    const randomWord = sillyWords[Math.floor(Math.random() * sillyWords.length)];

    setTimeout(() => {
      emit("spin-end", randomWord!);
    }, 1300);
    setTimeout(() => {
      if (pointerRef.value) {
        pointerRef.value.style.transition = "transform 0.4s ease-in-out";
      }
      pointerDeg.value = 315;
    }, 2500);
  } else if (chosenJoke === "uzunAdam") {
    // uzun adam varsa ekleme
    if (!props.options.includes("Uzun Adam")) {
      props.options.push!("Uzun Adam");
    }

    isSpinning.value = true;
    const n = props.options.length;
    const anglePerSliceDeg = 360 / n;

    const targetIndex = props.options.indexOf("Uzun Adam");
    const maxOffset = anglePerSliceDeg / 2 - 1;
    const wobble = (Math.random() * 2 - 1) * maxOffset;
    const targetDeg = -(targetIndex * anglePerSliceDeg) - 45 + wobble;
    const extraSpins = 4 + Math.floor(Math.random() * 3);
    const totalRotation = extraSpins * 360 + targetDeg;
    animState.startDeg = rotationDeg.value;
    animState.endDeg = totalRotation;
    animState.startTime = performance.now();
    animState.duration = 4200;
  }
}

// ---------- lifecycle ----------
onMounted(() => {
  drawWheel();
  if (rafId === null) {
    rafId = requestAnimationFrame(animationLoop);
  }
});

onBeforeUnmount(() => {
  if (rafId !== null) {
    cancelAnimationFrame(rafId);
  }
});

watch(
  () => props.options,
  () => {
    drawWheel();
  },
  { deep: true }
);
</script>

<style scoped></style>

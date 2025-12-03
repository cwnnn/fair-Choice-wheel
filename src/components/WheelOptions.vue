<template>
  <div class="w-full max-w-xl mx-auto">
    <!-- Başlık -->
    <div class="m-3 p-3 text-center bg-white rounded-xl">
      <div class="text-lg font-semibold flex items-center justify-center gap-2">
        Çevire
        <svg
          aria-hidden="true"
          focusable="false"
          data-prefix="fas"
          data-icon="circle-play"
          class="w-6 h-6"
          viewBox="0 0 512 512"
        >
          <path
            fill="currentColor"
            d="M512 256c0 141.4-114.6 256-256 256S0 397.4 0 256S114.6 0 256 0S512 114.6 512 256zM188.3 147.1c-7.6 4.2-12.3 12.3-12.3 20.9V344c0 8.7 4.7 16.7 12.3 20.9s16.8 4.1 24.3-.5l144-88c7.1-4.4 11.5-12.1 11.5-20.5s-4.4-16.1-11.5-20.5l-144-88c-7.4-4.5-16.7-4.7-24.3-.5z"
          />
        </svg>
        bas ve çarkı çevir!
      </div>
    </div>

    <!-- Seçenek sayısı -->
    <div class="m-3 p-3 text-center text-gray-700">Seçenek sayısı: {{ items.length }}</div>

    <!-- Toplu seçenek gir switch -->
    <div class="m-3 p-3 flex justify-between items-center">
      <div class="font-medium">Toplu Seçenek Gir</div>

      <label class="relative inline-flex items-center cursor-pointer">
        <input type="checkbox" class="sr-only peer" v-model="bulkMode" />
        <div
          class="w-12 h-6 bg-gray-300 peer-focus:outline-none rounded-full peer peer-checked:bg-blue-500 transition-all"
        ></div>
        <span
          class="absolute left-1 top-1 bg-white w-4 h-4 rounded-full shadow transition-all peer-checked:translate-x-6"
        >
        </span>
      </label>
    </div>

    <!-- Tekli input -->
    <div v-if="!bulkMode" class="m-3 p-3 flex">
      <input
        v-model="inputText"
        maxlength="30"
        placeholder="Çarka seçenek ekleyin"
        class="flex-1 border border-gray-300 rounded-lg rounded-r-none px-3 py-2 bg-white"
      />

      <button
        @click="addItem()"
        class="px-4 py-2 border rounded-lg border-gray-500 rounded-l-none hover:bg-gray-100"
      >
        Ekle
      </button>
    </div>

    <!-- Toplu input -->
    <div v-else class="m-3 p-3">
      <textarea
        v-model="bulkText"
        rows="5"
        placeholder="Her satıra bir seçenek"
        class="w-full border rounded-xl p-3"
      ></textarea>

      <button @click="addBulk()" class="mt-3 px-4 py-2 border rounded-lg hover:bg-gray-100 w-full">
        Toplu Ekle
      </button>
    </div>

    <!-- Liste -->
    <div class="m-3 p-3 bg-white rounded-xl shadow">
      <ul>
        <li
          v-for="(item, index) in items"
          :key="index"
          :class="[
            'flex justify-between items-center px-3 py-1 ',
            index % 2 === 0 ? 'bg-gray-100 rounded-xl' : 'bg-white',
          ]"
        >
          <span>{{ item }}</span>

          <svg
            @click="removeItem(index)"
            aria-hidden="true"
            data-prefix="far"
            data-icon="trash-can"
            class="w-4 h-4 cursor-pointer"
            viewBox="0 0 448 512"
          >
            <path
              fill="currentColor"
              d="M160 400C160 408.8 152.8 416 144 416C135.2 416
                     128 408.8 128 400V192C128 183.2 135.2 176 144 176C152.8
                     176 160 183.2 160 192V400zM240 400C240 408.8 232.8 416
                     224 416C215.2 416 208 408.8 208 400V192C208 183.2
                     215.2 176 224 176C232.8 176 240 183.2 240 192V400zM320
                     400C320 408.8 312.8 416 304 416C295.2 416 288 408.8
                     288 400V192C288 183.2 295.2 176 304 176C312.8 176 320
                     183.2 320 192V400zM317.5 24.94L354.2 80H424C437.3 80
                     448 90.75 448 104C448 117.3 437.3 128 424 128H416V432C416
                     476.2 380.2 512 336 512H112C67.82 512 32 476.2 32
                     432V128H24C10.75 128 0 117.3 0 104C0 90.75 10.75
                     80 24 80H93.82L130.5 24.94C140.9 9.357 158.4 0
                     177.1 0H270.9C289.6 0 307.1 9.358 317.5 24.94z"
            />
          </svg>
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch } from "vue";

const inputText = ref("");
const bulkText = ref("");
const bulkMode = ref(false);

const items = ref(["Seçenek 1", "Seçenek 2", "Seçenek 3", "Seçenek 4", "Seçenek 5", "Seçenek 6"]);

const emit = defineEmits<{
  (e: "update:options", value: string[]): void;
}>();

watch(
  items,
  (newItems) => {
    emit("update:options", newItems);
  },
  { deep: true }
);

function addItem() {
  if (!inputText.value.trim()) return;
  items.value.push(inputText.value.trim());
  inputText.value = "";
}

function addBulk() {
  const lines = bulkText.value
    .split("\n")
    .map((l) => l.trim())
    .filter((l) => l.length > 0);

  items.value.push(...lines);
  bulkText.value = "";
}

function removeItem(index: number) {
  items.value.splice(index, 1);
}
</script>

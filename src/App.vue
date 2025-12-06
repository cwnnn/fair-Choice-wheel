<script setup lang="ts">
import { RouterView } from "vue-router";
import Header from "./components/Header.vue";
</script>

<template>
  <div>
    <Header />
    <main class="flex-1">
      <router-view />
    </main>
  </div>
  <section class="w-full mx-auto pt-50 px-10 flex flex-col gap-28">
    <h1
      class="text-4xl md:text-5xl text-center mb-6 bg-gradient-to-r from-blue-600 to-black bg-clip-text text-transparent"
      style="font-weight: bold"
    >
      Çark Çevir - Karar Ver
    </h1>

    <div class="flex flex-col md:flex-row items-center gap-14">
      <div class="flex-1 text-center md:text-left">
        <h2 class="text-2xl md:text-3xl font-bold text-gray-800 mb-5">
          Karar Vermeyi Eğlenceli Hale Getirin
        </h2>
        <p class="text-gray-600 text-lg leading-relaxed mb-7">
          Kendi karar çarkınızı oluşturarak seçimlerinize heyecan katabilirsiniz.
        </p>
        <p class="text-gray-600 text-lg leading-relaxed mb-7">
          Eklediğiniz seçenekler arasından rastgele belirlenen sonuçlar, karar sürecini hem kolay
          hem de eğlenceli bir deneyime dönüştürür.
        </p>
      </div>

      <img src="/img1.png" class="w-44 h-44 md:w-52 md:h-52 object-contain flex-1" />
    </div>

    <div class="flex flex-col md:flex-row-reverse items-center gap-14">
      <div class="flex-1 text-center md:text-left">
        <h2 class="text-2xl md:text-3xl font-bold text-gray-800 mb-5">
          Seçeneklerinizi Kolayca Yönetin
        </h2>
        <p class="text-gray-600 text-lg leading-relaxed mb-7">
          Çarka eklediğiniz seçenekleri düzenleyebilir, toplu şekilde ekleyebilir veya gereksiz
          olanları tek dokunuşla kaldırabilirsiniz.
        </p>
        <p class="text-gray-600 text-lg leading-relaxed mb-7">
          Çark tamamen sizin tercihleriniz doğrultusunda şekillenir ve her kullanımda pratik bir
          deneyim sunar.
        </p>
      </div>

      <img src="/img2.png" class="w-44 h-44 md:w-52 md:h-52 object-contain flex-1" />
    </div>

    <div class="flex flex-col md:flex-row items-center gap-14">
      <div class="flex-1 text-center md:text-left">
        <h2 class="text-2xl md:text-3xl font-bold text-gray-800 mb-5">
          Kazananı Şık Bir Animasyonla Görün
        </h2>
        <p class="text-gray-600 text-lg leading-relaxed mb-7">
          Çark döndüğünde sonuç tamamen rastgele belirlenir ve ekranda özel bir animasyonla
          gösterilir.
        </p>
        <p class="text-gray-600 text-lg leading-relaxed mb-7">
          Böylece hem dikkat çekici hem de net bir sonuç elde etmiş olursunuz.
        </p>
      </div>

      <img src="/img3.png" class="w-44 h-44 md:w-52 md:h-52 object-contain flex-1" />
    </div>
  </section>
</template>

<style scoped></style>

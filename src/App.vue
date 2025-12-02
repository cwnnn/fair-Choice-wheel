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
</template>

<style scoped></style>

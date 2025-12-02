<script lang="ts" setup>
defineOptions({
  name: "AppHeader",
});
</script>

<template>
  <header class="header">
    <nav class="nav-container">
      <!-- Logo -->
      <a href="/" class="logo">
        <img src="../assets/logo.png" alt="logo" />
      </a>

      <!-- Menü -->
      <ul class="menu">
        <li><a href="/ne-yesem">Ne Yesem?</a></li>
        <li><a href="/sayi-carki">Sayı Çarkı</a></li>
        <li><a href="/tum-carklar">Tüm Çarklar</a></li>
      </ul>
    </nav>
  </header>
</template>

<style scoped>
@reference "tailwindcss/theme.css";
.header {
  @apply w-full border-b border-gray-300 pb-4;
}

.nav-container {
  @apply flex items-center justify-between max-w-[1200px] mx-auto w-full;
  padding: 0;
}

.logo {
  @apply flex items-center;
}

.menu {
  @apply flex gap-6 list-none m-0 p-0;
}

.menu li a {
  @apply font-bold text-blue-900 transition-colors duration-500 ease-in-out;
  text-decoration: none;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.menu li a:hover {
  @apply text-blue-700;
}

@media (max-width: 768px) {
  .nav-container {
    @apply flex-col gap-4;
  }
  .menu {
    @apply flex-col gap-2;
  }
}
</style>

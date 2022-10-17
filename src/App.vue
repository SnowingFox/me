<script setup lang="ts">
useHead({
  meta: [
    { property: 'og:title', content: 'Snowingfox' },
    { name: 'description', content: 'Snowingfox\'s Portfolio' },
    { name: 'twitter:card', content: 'summary' },
    { name: 'twitter:creator', content: '@FoxSnowing' },
  ],
})
</script>

<template>
  <NavBar />
  <main class="px-7 py-10">
    <router-view />
    <Footer />
  </main>
</template>

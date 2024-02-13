<template>
  <main
    class="
      h-full w-full
      flex justify-start items-center flex-col
      text-slate-700
    "
  >
    <nav-bar></nav-bar>

    <main-content></main-content>
  </main>
</template>

<script setup lang="ts">
import NavBar from 'src/components/nav-bar.vue';
import MainContent from 'src/components/main-content.vue';
</script>

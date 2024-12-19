<script setup>
// import { Icon } from '@iconify/vue'
import LinkCollection from '../components/LinkCollection.vue'
import SearchBox from '../components/SearchBox.vue'
</script>

<template>
  <div class="w-screen h-screen grid place-content-center">
    <div class="flex flex-col gap-12">
      <SearchBox />
      <LinkCollection />
    </div>
  </div>
</template>

<script setup>
// import { Icon } from '@iconify/vue'
import LinkCollection from '../components/LinkCollection.vue'
import SearchBox from '../components/SearchBox.vue'
</script>

<template>
  <div class="w-screen h-screen grid place-content-center">
    <div>
      <SearchBox />
      <LinkCollection />
    </div>
  </div>
</template>

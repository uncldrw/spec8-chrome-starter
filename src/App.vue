<script setup>
// import { Icon } from '@iconify/vue'
import { onMounted } from 'vue'
import LinkCollection from '../components/LinkCollection.vue'
import SearchBox from '../components/SearchBox.vue'
import 'css-houdini-squircle'

onMounted(async () => {
  if (!CSS.paintWorklet) {
    await import('css-paint-polyfill')
  }
  await CSS.paintWorklet.addModule('https://unpkg.com/css-houdini-squircle/squircle.min.js')
})
</script>

<template>
  <div class="w-screen h-screen grid place-content-center">
    <div class="flex flex-col gap-12">
      <SearchBox />
      <LinkCollection />
    </div>
    <span></span>
    <span></span>
    <span></span>
  </div>
</template>

<script></script>

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
  <div class="w-screen h-screen flex mt">
    <div class="flex flex-col gap-12 w-full mt-[20vh]">
      <h1 class="text-7xl font-bold text-white text-center mb-4">Google</h1>
      <SearchBox />
      <LinkCollection />
    </div>
    <span></span>
    <span></span>
    <span></span>
  </div>
</template>

<style>
html {
  interpolate-size: allow-keywords;
}
</style>

<script setup lang="ts">
import { Icon } from '@iconify/vue'
</script>

<template>
  <div class="max-w-2xl mx-auto w-full">
    <span class="bg-neutral-100 w-full inline-flex relative border border-neutral-200 rounded-xl">
      <span
        class="size-8 inline-grid place-content-center z-10 absolute left-2 top-1/2 -translate-y-1/2 rounded"
      >
        <Icon icon="f7:search" class="text-neutral-400 text-2xl" />
      </span>
      <input
        type="text"
        class="bg-transparent focus:outline-none w-full pl-12 py-3 pr-12 text-neutral-950"
      />
      <span
        class="size-8 bg-teal-600 inline-grid place-content-center z-10 absolute right-2 top-1/2 -translate-y-1/2 rounded-md"
      >
        <Icon icon="f7:arrow-up-right" class="text-white text-2xl" />
      </span>
    </span>
  </div>
</template>

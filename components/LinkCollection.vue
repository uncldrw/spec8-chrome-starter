<script setup>
import draggable from 'vuedraggable'
import { ref, computed, onMounted } from 'vue'
import ColorThief from 'colorthief'

const links = ref([
  { id: 1, name: 'Google', url: 'https://google.com' },
  { id: 2, name: 'Facebook', url: 'https://facebook.com' },
  { id: 3, name: 'Twitter', url: 'https://twitter.com' },
  { id: 4, name: 'Instagram', url: 'https://instagram.com' },
  { id: 5, name: 'LinkedIn', url: 'https://linkedin.com' },
  { id: 6, name: 'YouTube', url: 'https://youtube.com' },
  { id: 7, name: 'Reddit', url: 'https://reddit.com' },
  { id: 8, name: 'Pinterest', url: 'https://pinterest.com' },
  { id: 9, name: 'GitHub', url: 'https://github.com' },
  { id: 10, name: 'Stack Overflow', url: 'https://stackoverflow.com' },
  { id: 11, name: 'Wikipedia', url: 'https://wikipedia.org' },
  { id: 13, name: 'Netflix', url: 'https://netflix.com' },
  { id: 14, name: 'Spotify', url: 'https://spotify.com' },
  { id: 15, name: 'Twitch', url: 'https://twitch.tv' },
  { id: 16, name: 'WhatsApp', url: 'https://whatsapp.com' },
  { id: 17, name: 'TikTok', url: 'https://tiktok.com' },
  { id: 18, name: 'Snapchat', url: 'https://snapchat.com' },
  { id: 19, name: 'Discord', url: 'https://discord.com' },
  { id: 21, name: 'Zoom', url: 'https://zoom.us' },
  { id: 22, name: 'Microsoft', url: 'https://microsoft.com' },
  { id: 23, name: 'Apple', url: 'https://apple.com' },
])

const list = links.value.map((entry, index) => {
  return { ...entry, order: index + 1 }
})

const drag = ref(false)

const dragOptions = computed(() => {
  return {
    animation: 300,
    group: 'description',
    disabled: false,
    ghostClass: 'ghost',
    forceFallback: true,
    fallbackClass: 'sortable-fallback',
  }
})

const getAverageColor = (url) => {
  return new Promise((resolve, reject) => {
    const colorThief = new ColorThief()
    const img = new Image()

    img.addEventListener('load', function () {
      try {
        const [r, g, b] = colorThief.getPalette(img)[2]
        const val = `rgba(${r},${g},${b}, 0.4)`

        resolve(val)
      } catch (error) {
        reject(error)
      }
    })

    img.addEventListener('error', function (error) {
      reject(error)
    })

    let proxy = 'https://favicone.com/'
    img.crossOrigin = 'Anonymous'
    img.src = proxy + encodeURIComponent(url.split('https://')[1]) + '?s=32'
  })
}

const linkColors = ref({})

onMounted(async () => {
  for (const link of list) {
    try {
      linkColors.value[link.url] = await getAverageColor(link.url)
    } catch (error) {
      linkColors.value[link.url] = '#666666' // Fallback color
      console.log(error)
    }
  }
})

const getColorByUrl = (url) => {
  return linkColors.value[url] || '#666666'
}
</script>

<template>
  <div>
    <draggable
      class="flex flex-wrap justify-center gap-4 max-w-screen-2xl"
      :component-data="{ type: 'transition-group', name: !drag ? 'flip-list' : null }"
      item-key="order"
      tag="ul"
      v-bind="dragOptions"
      v-model="list"
      @start="drag = true"
      @end="drag = false"
    >
      <template #item="{ element }">
        <li class="card relative">
          <div
            class="size-24 select-none squircle"
            @click="element.fixed = !element.fixed"
            :style="`background-color: ${getColorByUrl(element.url)};`"
          >
            <a
              :href="element.url"
              target="_blank"
              class="w-full h-full flex flex-col items-center justify-center"
            >
              <span class="size-12 rounded-full inline-grid place-content-center">
                <img
                  class="size-8 rounded"
                  :src="`https://favicone.com/${element.url.split('https://')[1]}?s=64`"
                  alt=""
                />
              </span>
              <p class="text-xs text-center text-gray-100 font-medium">{{ element.name }}</p>
            </a>
          </div>
        </li>
      </template>
    </draggable>
  </div>
</template>

<style>
.squircle {
  --squircle-smooth: 0.4;
  --squircle-radius: 14;
  mask-image: paint(squircle);
}

.card:after {
  content: '';
  @apply w-full h-full absolute top-0 left-0 -z-10 bg-neutral-600;
  --squircle-smooth: 0.4;
  --squircle-radius: 14;
  mask-image: paint(squircle);
}

.sortable-chosen {
  cursor: move !important;
}

.ghost {
  opacity: 0;
}
.button {
  margin-top: 35px;
}

.flip-list-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.list-group {
  min-height: 20px;
}

.list-group-item {
  cursor: move;
}

.list-group-item i {
  cursor: pointer;
}
</style>

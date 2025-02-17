<!-- eslint-disable no-unused-vars -->
<script setup>
import draggable from 'vuedraggable'
import { ref, computed, onMounted } from 'vue'
import ColorThief from 'colorthief'
import { Icon } from '@iconify/vue'

const DB_NAME = 'linksDB'
const STORE_NAME = 'links'
const db = ref(null)
const linkColors = ref({})
const drag = ref(false)

const list = ref([])

// Initialisiere IndexedDB
const initDB = () => {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open(DB_NAME, 1)

    request.onerror = () => reject(request.error)
    request.onsuccess = () => {
      db.value = request.result
      resolve(db.value)
    }

    request.onupgradeneeded = (event) => {
      const db = event.target.result
      if (!db.objectStoreNames.contains(STORE_NAME)) {
        db.createObjectStore(STORE_NAME, { keyPath: 'id' })
      }
    }
  })
}

// Speichere Links in IndexedDB
const saveLinks = async () => {
  const transaction = db.value.transaction(STORE_NAME, 'readwrite')
  const store = transaction.objectStore(STORE_NAME)

  // Links mit aktualisierter Reihenfolge speichern
  const linksToSave = list.value.map((link, index) => ({
    ...link,
    order: index + 1,
    color: linkColors.value[link.url],
  }))

  for (const link of linksToSave) {
    await store.put(link)
  }
}

// Lade Links aus IndexedDB
const loadLinks = async () => {
  const transaction = db.value.transaction(STORE_NAME, 'readonly')
  const store = transaction.objectStore(STORE_NAME)

  return new Promise((resolve, reject) => {
    const request = store.getAll()
    request.onerror = () => reject(request.error)
    request.onsuccess = () => {
      const sortedLinks = request.result.sort((a, b) => a.order - b.order)
      resolve(sortedLinks)
    }
  })
}

// Modifiziere onMounted
onMounted(async () => {
  await initDB()

  // Prüfe ob Links in IndexedDB existieren
  const savedLinks = await loadLinks()

  if (savedLinks.length > 0) {
    // Wenn ja, verwende die gespeicherten Links und Farben
    list.value.splice(0, list.value.length, ...savedLinks)
    savedLinks.forEach((link) => {
      linkColors.value[link.url] = link.color
    })
  } else {
    // Wenn nein, lade Farben für Standardlinks
    for (const link of list.value) {
      try {
        linkColors.value[link.url] = await getAverageColor(link.url)
      } catch (error) {
        linkColors.value[link.url] = '#666666'
        console.log(error)
      }
    }
    // Speichere die neuen Links
    await saveLinks()
  }
})

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

const updateOrder = async () => {
  drag.value = false
  // Aktualisiere die Reihenfolge
  list.value.forEach((item, index) => {
    item.order = index + 1
  })
  // Speichere sofort
  await saveLinks()
}

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

onMounted(async () => {
  for (const link of list.value) {
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

const addLink = async () => {
  // eslint-disable-next-line no-unused-vars
  const mockLinks = ['test.de', 'google.com', 'youtube.com', 'facebook.com']
  const name = `${Date.now()}`
  const url = mockLinks[3]
  if (!name || !url) return

  try {
    const formattedUrl = url.startsWith('https://') ? url : `https://${url}`
    const color = await getAverageColor(formattedUrl)

    const newLink = {
      id: Date.now(),
      type: 'link',
      name,
      url: formattedUrl,
      order: list.value.length + 1,
      color,
    }

    list.value.push(newLink)
    linkColors.value[formattedUrl] = color
    await saveLinks()
  } catch (error) {
    console.error('Fehler beim Hinzufügen des Links:', error)
  }
}

const addFolder = () => {
  const newLink = {
    id: Date.now(),
    type: 'folder',
    name: 'Neuer Ordner',
    url: null,
    order: list.value.length + 1,
    open: false,
  }
  list.value.push(newLink)
}
</script>

<template>
  <div class="flex justify-center">
    <draggable
      class="w-full max-w-screen-xl flex flex-wrap gap-4 justify-center"
      :component-data="{ type: 'transition-group', name: 'move-list' }"
      item-key="id"
      tag="ul"
      v-bind="dragOptions"
      v-model="list"
      @start="drag = true"
      @end="updateOrder()"
    >
      <template #item="{ element }">
        <TransitionGroup :name="!drag ? 'flip-list' : 'move-list'" tag="div" class="inline-block">
          <li class="card relative" :key="element.id">
            <template v-if="element.type = 'folder'">
              <div
                class="h-24 select-none squircle transition-all duration-700"
                @click="element.open = !element.open"
                :class="{ 'w-24': !element.open, 'w-auto': element.open }"
              >
                <input type="radio" name="folder" />
                <Icon icon="fluent:folder-24-filled" class="size-8 text-gray-100"></Icon>
                <div class="w-[32rem] transition-all">asd</div>
              </div>
            </template>
            <template v-else>
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
                      v-if="element.url"
                      class="size-8 rounded"
                      :src="`https://favicone.com/${element.url.split('https://')[1]}?s=64`"
                      alt=""
                    />
                    <Icon v-else icon="fluent:folder-24-filled" class="size-8 text-gray-100"></Icon>
                  </span>
                  <p class="text-xs text-center text-gray-100 font-medium">{{ element.name }}</p>
                </a>
              </div>
            </template>
          </li>
        </TransitionGroup>
      </template>
      <template #footer>
        <div class="size-24 select-none grid place-content-center">
          <Icon
            @click="addFolder"
            icon="f7:plus-app-fill"
            class="size-10 text-neutral-700 hover:text-neutral-600 cursor-pointer"
          />
        </div>
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
  transition: all 0.7s;
}

.move-list-enter-active,
.move-list-leave-active {
  transition: all 0.7s;
}

.move-list-leave-active {
  position: absolute;
}
.move-list-enter-from,
.move-list-leave-to {
  opacity: 0;
}

.flip-list-leave-active {
  position: absolute;
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

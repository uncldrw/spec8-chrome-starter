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
  const name = 'Test'
  const url = 'test.de'
  if (!name || !url) return

  try {
    const formattedUrl = url.startsWith('https://') ? url : `https://${url}`
    const color = await getAverageColor(formattedUrl)

    const newLink = {
      id: Date.now(),
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
</script>

<template>
  <div class="flex justify-center">
    <draggable
      class="flex flex-wrap justify-center gap-4 max-w-screen-xl"
      :component-data="{ animation: 75 }"
      item-key="id"
      tag="ul"
      v-bind="dragOptions"
      v-model="list"
      @start="drag = true"
      @end="updateOrder()"
    >
      <template #item="{ element }">
        <TransitionGroup name="flip-list" tag="div">
          <li class="card relative" :key="element.id">
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
        </TransitionGroup>
      </template>
      <template #footer>
        <div class="size-24 select-none grid place-content-center">
          <Icon
            @click="addLink"
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

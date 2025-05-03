<script setup>
import { Analytics } from '@vercel/analytics/next'
import { ref, computed } from 'vue'

// Array data
const items = ref([])

// Input
const newItem = ref('')

// Pilihan status filter: all / done / not_done
const filterStatus = ref('all')

// Mobile view state
const isMobileMenuOpen = ref(false)

// Tambah item baru
const addItem = () => {
  if (newItem.value.trim() !== '') {
    items.value.push({
      id: Date.now(),
      name: newItem.value,
      done: false
    })
    newItem.value = ''
    // Close mobile menu after adding on mobile devices
    isMobileMenuOpen.value = false
  }
}

// Toggle status done
const toggleDone = (item) => {
  item.done = !item.done
}

// Computed untuk filter berdasarkan status
const filteredItems = computed(() => {
  if (filterStatus.value === 'done') {
    return items.value.filter(item => item.done)
  } else if (filterStatus.value === 'not_done') {
    return items.value.filter(item => !item.done)
  } else {
    return items.value
  }
})

const deleteItem = (item) => {
  const idx = items.value.findIndex(i => i.id === item.id)
  if (idx !== -1) {
    items.value.splice(idx, 1)
  }
}

// Counter for completed tasks
const completedCount = computed(() => {
  return items.value.filter(item => item.done).length
})

// Counter for remaining tasks
const remainingCount = computed(() => {
  return items.value.filter(item => !item.done).length
})

// Toggle mobile menu
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value
}
</script>

<template>
  <!-- Main container with dark mode theme -->
  <div class="h-screen w-screen overflow-hidden bg-gray-900 text-gray-100 font-sans flex flex-col">
    <!-- Sidebar Header -->
    <div class="w-full bg-gray-800 py-4 px-4 md:py-6 md:px-8 flex items-center justify-between">
      <div class="flex items-center">
        <div class="w-8 h-8 md:w-10 md:h-10 rounded-lg bg-teal-500 flex items-center justify-center mr-3 md:mr-4">
          <span class="text-lg md:text-xl font-bold">✓</span>
        </div>
        <h1 class="text-xl md:text-2xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-teal-400 to-blue-500">TaskFlow</h1>
      </div>

      <!-- Mobile menu button -->
      <button @click="toggleMobileMenu" class="md:hidden bg-gray-700 rounded-lg p-2 text-gray-300 hover:bg-gray-600">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>

      <!-- Task counters - visible on desktop only -->
      <div class="hidden md:flex items-center">
        <div class="mr-6 bg-gray-700 rounded-full px-4 py-1 text-sm flex items-center">
          <span class="mr-2 text-teal-400">{{ completedCount }}</span> selesai
        </div>
        <div class="bg-gray-700 rounded-full px-4 py-1 text-sm flex items-center">
          <span class="mr-2 text-blue-400">{{ remainingCount }}</span> tersisa
        </div>
      </div>
    </div>

    <!-- Main content area -->
    <div class="flex-1 flex flex-col md:flex-row overflow-hidden">
      <!-- Mobile sidebar - conditionally rendered based on state -->
      <div v-if="isMobileMenuOpen" class="md:hidden bg-gray-800 p-4 border-b border-gray-700 flex flex-col">
        <div class="mb-4">
          <h2 class="text-lg font-semibold mb-3 text-teal-400">Buat Task Baru</h2>
          <div class="bg-gray-700 rounded-lg p-1 mb-3">
            <input v-model="newItem" @keyup.enter="addItem" placeholder="Apa yang perlu dilakukan?"
              class="w-full bg-gray-700 border-0 text-gray-100 p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500 transition-all duration-300 placeholder-gray-400" />
          </div>
          <button @click="addItem"
            class="w-full bg-gradient-to-r from-teal-500 to-blue-500 hover:from-teal-600 hover:to-blue-600 text-white py-2 px-4 rounded-lg transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-teal-500 focus:ring-opacity-50 font-medium">
            Tambahkan Task
          </button>
        </div>

        <div class="mb-4">
          <h2 class="text-lg font-semibold mb-3 text-blue-400">Filter</h2>
          <div class="grid grid-cols-3 gap-2">
            <button @click="filterStatus = 'all'; toggleMobileMenu()"
              class="py-2 px-2 rounded-lg transition-all duration-300 text-center text-sm font-medium"
              :class="filterStatus === 'all' ? 'bg-gray-600 text-white' : 'bg-gray-700 text-gray-400 hover:bg-gray-600'">
              Semua
            </button>
            <button @click="filterStatus = 'done'; toggleMobileMenu()"
              class="py-2 px-2 rounded-lg transition-all duration-300 text-center text-sm font-medium"
              :class="filterStatus === 'done' ? 'bg-teal-600 text-white' : 'bg-gray-700 text-gray-400 hover:bg-gray-600'">
              Selesai
            </button>
            <button @click="filterStatus = 'not_done'; toggleMobileMenu()"
              class="py-2 px-2 rounded-lg transition-all duration-300 text-center text-sm font-medium"
              :class="filterStatus === 'not_done' ? 'bg-blue-600 text-white' : 'bg-gray-700 text-gray-400 hover:bg-gray-600'">
              Aktif
            </button>
          </div>
        </div>

        <!-- Mobile counters -->
        <div class="flex justify-between mb-2">
          <div class="bg-gray-700 rounded-full px-3 py-1 text-xs flex items-center">
            <span class="mr-1 text-teal-400">{{ completedCount }}</span> selesai
          </div>
          <div class="bg-gray-700 rounded-full px-3 py-1 text-xs flex items-center">
            <span class="mr-1 text-blue-400">{{ remainingCount }}</span> tersisa
          </div>
        </div>
      </div>

      <!-- Left sidebar for input - hidden on mobile unless toggled -->
      <div class="hidden md:block md:w-1/3 lg:w-1/4 bg-gray-800 p-4 md:p-6 flex-col border-r border-gray-700">
        <div class="mb-6">
          <h2 class="text-lg font-semibold mb-4 text-teal-400">Buat Task Baru</h2>
          <div class="bg-gray-700 rounded-lg p-1 mb-4">
            <input v-model="newItem" @keyup.enter="addItem" placeholder="Apa yang perlu dilakukan?"
              class="w-full bg-gray-700 border-0 text-gray-100 p-3 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500 transition-all duration-300 placeholder-gray-400" />
          </div>
          <button @click="addItem"
            class="w-full bg-gradient-to-r from-teal-500 to-blue-500 hover:from-teal-600 hover:to-blue-600 text-white py-3 px-4 rounded-lg transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-teal-500 focus:ring-opacity-50 font-medium">
            Tambahkan Task
          </button>
        </div>

        <div>
          <h2 class="text-lg font-semibold mb-4 text-blue-400">Filter</h2>
          <div class="grid grid-cols-3 gap-2">
            <button @click="filterStatus = 'all'"
              class="py-2 px-4 rounded-lg transition-all duration-300 text-center font-medium"
              :class="filterStatus === 'all' ? 'bg-gray-600 text-white' : 'bg-gray-700 text-gray-400 hover:bg-gray-600'">
              Semua
            </button>
            <button @click="filterStatus = 'done'"
              class="py-2 px-4 rounded-lg transition-all duration-300 text-center font-medium"
              :class="filterStatus === 'done' ? 'bg-teal-600 text-white' : 'bg-gray-700 text-gray-400 hover:bg-gray-600'">
              Selesai
            </button>
            <button @click="filterStatus = 'not_done'"
              class="py-2 px-4 rounded-lg transition-all duration-300 text-center font-medium"
              :class="filterStatus === 'not_done' ? 'bg-blue-600 text-white' : 'bg-gray-700 text-gray-400 hover:bg-gray-600'">
              Aktif
            </button>
          </div>
        </div>
      </div>

      <!-- Right content area for list -->
      <div class="flex-1 p-4 md:p-6 overflow-hidden flex flex-col">
        <div class="flex items-center justify-between mb-4 md:mb-6">
          <h2 class="text-xl md:text-2xl font-bold">
            <span v-if="filterStatus === 'all'">Semua Task</span>
            <span v-else-if="filterStatus === 'done'" class="text-teal-400">Task Selesai</span>
            <span v-else class="text-blue-400">Task Aktif</span>
          </h2>

          <div class="text-xs md:text-sm text-gray-400">
            {{ filteredItems.length }} task ditampilkan
          </div>
        </div>

        <!-- Add new task quick button - mobile only -->
        <button v-if="!isMobileMenuOpen" @click="toggleMobileMenu" 
          class="md:hidden mb-4 bg-gradient-to-r from-teal-500 to-blue-500 hover:from-teal-600 hover:to-blue-600 text-white py-2 px-4 rounded-lg flex items-center justify-center transition-all duration-300">
          <span class="mr-2">+</span> Tambah Task
        </button>

        <!-- Scrollable list container -->
        <div class="flex-1 overflow-y-auto pr-2 md:pr-4 -mr-2 md:-mr-4">
          <!-- Empty state -->
          <div v-if="filteredItems.length === 0"
            class="h-40 md:h-64 flex flex-col items-center justify-center py-6 md:py-8 text-center text-gray-500">
            <div class="w-12 h-12 md:w-16 md:h-16 mb-3 md:mb-4 rounded-full bg-gray-800 flex items-center justify-center">
              <span class="text-xl md:text-2xl">📋</span>
            </div>
            <p class="text-base md:text-lg">Tidak ada task untuk ditampilkan</p>
            <p class="text-xs md:text-sm mt-2" v-if="filterStatus !== 'all'">Coba ubah filter atau tambahkan task baru</p>
          </div>

          <!-- Task list -->
          <ul class="space-y-3 md:space-y-4">
            <li v-for="item in filteredItems" :key="item.id"
              class="bg-gray-800 rounded-xl overflow-hidden transition-all duration-300 transform hover:-translate-y-1">
              <div class="p-3 md:p-4 flex items-center justify-between">
                <div class="flex items-center truncate mr-2">
                  <button @click="toggleDone(item)"
                    class="min-w-6 w-6 h-6 rounded-full mr-3 md:mr-4 flex items-center justify-center transition-all duration-300 border-2 flex-shrink-0"
                    :class="item.done ? 'bg-teal-500 border-transparent' : 'border-gray-500 hover:border-teal-500'">
                    <span v-if="item.done" class="text-white text-xs">✓</span>
                  </button>

                  <span class="text-base md:text-lg transition-all duration-300 truncate"
                    :class="{ 'line-through text-gray-500': item.done, 'text-white': !item.done }">
                    {{ item.name }}
                  </span>
                </div>

                <div class="flex items-center space-x-2 flex-shrink-0">
                  <div v-if="item.done" class="hidden sm:block px-3 py-1 rounded-full bg-teal-900 text-teal-300 text-xs">
                    Selesai
                  </div>
                  <div v-else class="hidden sm:block px-3 py-1 rounded-full bg-blue-900 text-blue-300 text-xs">
                    Aktif
                  </div>

                  <button @click="deleteItem(item)"
                    class="w-8 h-8 rounded-full text-gray-400 hover:text-red-400 hover:bg-gray-700 flex items-center justify-center transition-all duration-300">
                    ×
                  </button>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <Analytics />
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700&display=swap');

* {
  font-family: 'Outfit', sans-serif;
}

/* Animasi untuk item baru */
li {
  animation: slideIn 0.4s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* Custom scrollbar */
::-webkit-scrollbar {
  width: 4px;
}

::-webkit-scrollbar-track {
  background: #2d3748;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background: #4fd1c5;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: #38b2ac;
}

/* Pulse animation for empty state */
@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.05);
  }
}

/* Button hover effects */
button:active {
  transform: scale(0.95);
}

/* Task completion animation */
.task-complete-animation {
  animation: completeTask 0.5s ease-in-out;
}

@keyframes completeTask {
  0% {
    background-color: transparent;
  }

  50% {
    background-color: rgba(79, 209, 197, 0.2);
  }

  100% {
    background-color: transparent;
  }
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .truncate {
    max-width: 200px;
  }
}

@media (max-width: 640px) {
  .truncate {
    max-width: 160px;
  }
}

@media (max-width: 480px) {
  .truncate {
    max-width: 120px;
  }
}
</style>
<script setup>
import { RouterLink, useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { ref } from 'vue'

const authStore = useAuthStore()
const router = useRouter()
const sidebarOpen = ref(true)

const menuItems = [
  { name: 'Dashboard', path: '/dashboard', icon: '📊' },
  { name: 'POS / Kasir', path: '/pos', icon: '🛒' },
  { name: 'Products', path: '/products', icon: '📦' },
  { name: 'Categories', path: '/categories', icon: '🏷️' },
  { name: 'Transactions', path: '/transactions', icon: '💰' },
]

async function handleLogout() {
  await authStore.logout()
  router.push('/login')
}
</script>

<template>
  <div class="min-h-screen bg-gray-50 flex">

    <!-- Sidebar -->
    <aside
      :class="sidebarOpen ? 'w-64' : 'w-20'"
      class="bg-white shadow-lg transition-all duration-300 flex flex-col"
    >
      <!-- Logo -->
      <div class="p-4 border-b border-gray-200 flex items-center justify-between">
        <div v-if="sidebarOpen" class="flex items-center gap-2">
          <span class="text-2xl">🛒</span>
          <span class="font-bold text-lg text-gray-800">Mini POS</span>
        </div>
        <span v-else class="text-2xl mx-auto">🛒</span>
        <button
          @click="sidebarOpen = !sidebarOpen"
          class="text-gray-500 hover:text-gray-800"
          v-if="sidebarOpen"
        >
          ◀
        </button>
      </div>

      <!-- Toggle Open (jika tertutup) -->
      <button
        v-if="!sidebarOpen"
        @click="sidebarOpen = true"
        class="p-2 text-gray-500 hover:text-gray-800 text-center"
      >
        ▶
      </button>

      <!-- Menu -->
      <nav class="flex-1 p-3 space-y-1">
        <RouterLink
          v-for="item in menuItems"
          :key="item.path"
          :to="item.path"
          class="flex items-center gap-3 px-3 py-2.5 rounded-lg text-gray-700 hover:bg-primary-50 hover:text-primary-700 transition"
          active-class="bg-primary-100 text-primary-700 font-semibold"
        >
          <span class="text-xl">{{ item.icon }}</span>
          <span v-if="sidebarOpen">{{ item.name }}</span>
        </RouterLink>
      </nav>

      <!-- User Info -->
      <div class="p-3 border-t border-gray-200">
        <div v-if="sidebarOpen" class="mb-2 px-2">
          <p class="text-sm font-semibold text-gray-800">{{ authStore.user?.name }}</p>
          <p class="text-xs text-gray-500 capitalize">{{ authStore.user?.role }}</p>
        </div>
        <button
          @click="handleLogout"
          class="w-full flex items-center gap-2 px-3 py-2 text-red-600 hover:bg-red-50 rounded-lg transition"
        >
          <span class="text-xl">🚪</span>
          <span v-if="sidebarOpen">Logout</span>
        </button>
      </div>
    </aside>

    <!-- Main Content -->
    <main class="flex-1 overflow-auto">
      <slot />
    </main>

  </div>
</template>

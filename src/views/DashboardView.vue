<script setup>
import { ref, onMounted } from 'vue'
import AppLayout from '@/components/AppLayout.vue'
import api from '@/services/api'

const stats = ref(null)
const loading = ref(true)

function formatRupiah(amount) {
  return 'Rp ' + Number(amount).toLocaleString('id-ID')
}

onMounted(async () => {
  try {
    const response = await api.get('/dashboard/stats')
    if (response.data.success) {
      stats.value = response.data.data
    }
  } catch (error) {
    console.error('Failed to fetch stats:', error)
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <AppLayout>
    <div class="p-8">
      <h1 class="text-3xl font-bold text-gray-800 mb-2">Dashboard 📊</h1>
      <p class="text-gray-500 mb-8">Statistik bisnis kamu hari ini</p>

      <!-- Loading -->
      <div v-if="loading" class="text-center py-12 text-gray-500">
        Loading stats...
      </div>

      <!-- Stats Cards -->
      <div v-else-if="stats">
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
          
          <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <div class="flex items-center justify-between mb-2">
              <p class="text-sm text-gray-500">Total Revenue</p>
              <span class="text-2xl">💰</span>
            </div>
            <p class="text-2xl font-bold text-gray-800">{{ formatRupiah(stats.total_revenue) }}</p>
          </div>

          <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <div class="flex items-center justify-between mb-2">
              <p class="text-sm text-gray-500">Total Orders</p>
              <span class="text-2xl">🧾</span>
            </div>
            <p class="text-2xl font-bold text-gray-800">{{ stats.total_orders }}</p>
          </div>

          <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <div class="flex items-center justify-between mb-2">
              <p class="text-sm text-gray-500">Total Products</p>
              <span class="text-2xl">📦</span>
            </div>
            <p class="text-2xl font-bold text-gray-800">{{ stats.total_products }}</p>
          </div>

          <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <div class="flex items-center justify-between mb-2">
              <p class="text-sm text-gray-500">Low Stock</p>
              <span class="text-2xl">⚠️</span>
            </div>
            <p class="text-2xl font-bold text-red-600">{{ stats.low_stock_products }}</p>
          </div>

        </div>

        <!-- Top Products & Recent Transactions -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
          
          <!-- Top Products -->
          <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <h2 class="text-lg font-semibold text-gray-800 mb-4">🏆 Top Products</h2>
            <div v-if="stats.top_products.length === 0" class="text-gray-400 text-sm text-center py-8">
              Belum ada penjualan
            </div>
            <div v-else class="space-y-3">
              <div
                v-for="(product, index) in stats.top_products"
                :key="index"
                class="flex items-center justify-between py-2 border-b border-gray-100 last:border-0"
              >
                <div>
                  <p class="font-medium text-gray-800">{{ product.product_name }}</p>
                  <p class="text-xs text-gray-500">{{ product.total_sold }} terjual</p>
                </div>
                <p class="font-semibold text-primary-600">{{ formatRupiah(product.total_revenue) }}</p>
              </div>
            </div>
          </div>

          <!-- Recent Transactions -->
          <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <h2 class="text-lg font-semibold text-gray-800 mb-4">🧾 Recent Transactions</h2>
            <div v-if="stats.recent_transactions.length === 0" class="text-gray-400 text-sm text-center py-8">
              Belum ada transaksi
            </div>
            <div v-else class="space-y-3">
              <div
                v-for="trx in stats.recent_transactions"
                :key="trx.id"
                class="flex items-center justify-between py-2 border-b border-gray-100 last:border-0"
              >
                <div>
                  <p class="font-medium text-gray-800 text-sm">{{ trx.invoice_number }}</p>
                  <p class="text-xs text-gray-500">{{ trx.user?.name }}</p>
                </div>
                <p class="font-semibold text-green-600">{{ formatRupiah(trx.total_amount) }}</p>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>
  </AppLayout>
</template>
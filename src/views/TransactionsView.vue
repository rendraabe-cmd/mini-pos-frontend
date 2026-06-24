<template>
  <AppLayout>
    <div class="p-6">

      <!-- Header -->
      <div class="flex items-center justify-between mb-6">
        <div>
          <h1 class="text-2xl font-bold text-gray-800">Transactions</h1>
          <p class="text-gray-500 text-sm mt-1">Riwayat semua transaksi penjualan</p>
        </div>
        <div class="text-right">
          <p class="text-xs text-gray-500">Total Transaksi</p>
          <p class="text-2xl font-bold text-primary-600">{{ transactions.length }}</p>
        </div>
      </div>

      <!-- Alert -->
      <div v-if="alert.show" :class="[
        'mb-4 px-4 py-3 rounded-lg text-sm font-medium',
        alert.type === 'success'
          ? 'bg-green-100 text-green-800 border border-green-200'
          : 'bg-red-100 text-red-800 border border-red-200'
      ]">
        {{ alert.message }}
      </div>

      <!-- Filter -->
      <div class="flex gap-3 mb-4">
        <input
          v-model="search"
          type="text"
          placeholder="🔍 Cari invoice atau kasir..."
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500 w-64"
        />
        <select
          v-model="filterPayment"
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500"
        >
          <option value="">Semua Pembayaran</option>
          <option value="cash">Cash</option>
          <option value="qris">QRIS</option>
        </select>
      </div>

      <!-- Loading -->
      <div v-if="loading" class="flex justify-center items-center py-20">
        <div class="animate-spin rounded-full h-10 w-10 border-b-2 border-primary-600"></div>
      </div>

      <!-- Table -->
      <div v-else class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
        <table class="w-full">
          <thead class="bg-gray-50 border-b border-gray-100">
            <tr>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Invoice</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Tanggal</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Kasir</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Items</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Total</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Pembayaran</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Aksi</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-50">
            <tr v-if="filteredTransactions.length === 0">
              <td colspan="7" class="text-center py-12 text-gray-400">
                <div class="text-4xl mb-2">💰</div>
                <p>Belum ada transaksi</p>
              </td>
            </tr>
            <tr
              v-for="trx in filteredTransactions"
              :key="trx.id"
              class="hover:bg-gray-50 transition"
            >
              <td class="px-6 py-4">
                <span class="font-mono text-sm font-semibold text-primary-600">
                  {{ trx.invoice_number ?? `#TRX${trx.id}` }}
                </span>
              </td>
              <td class="px-6 py-4 text-sm text-gray-600">
                {{ formatDate(trx.created_at) }}
              </td>
              <td class="px-6 py-4">
                <div class="flex items-center gap-2">
                  <div class="w-7 h-7 rounded-full bg-primary-100 flex items-center justify-center text-xs font-bold text-primary-600">
                    {{ (trx.user?.name ?? 'U').charAt(0).toUpperCase() }}
                  </div>
                  <span class="text-sm text-gray-700">{{ trx.user?.name ?? '-' }}</span>
                </div>
              </td>
              <td class="px-6 py-4 text-sm text-gray-600">
                {{ trx.items?.length ?? trx.transaction_items?.length ?? 0 }} item
              </td>
              <td class="px-6 py-4 text-sm font-bold text-gray-800">
                {{ formatCurrency(trx.total) }}
              </td>
              <td class="px-6 py-4">
                <span :class="[
                  'text-xs px-2 py-1 rounded-full font-medium uppercase',
                  trx.payment_method === 'cash'
                    ? 'bg-green-100 text-green-700'
                    : 'bg-purple-100 text-purple-700'
                ]">
                  {{ trx.payment_method }}
                </span>
              </td>
              <td class="px-6 py-4">
                <button
                  @click="viewDetail(trx)"
                  class="bg-primary-50 hover:bg-primary-100 text-primary-700 px-3 py-1.5 rounded-lg text-sm font-medium transition"
                >
                  👁️ Detail
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

    </div>

    <!-- Modal Detail -->
    <div v-if="detailModal.show" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
      <div class="bg-white rounded-2xl shadow-2xl w-full max-w-md max-h-[90vh] overflow-y-auto">

        <!-- Header -->
        <div class="flex items-center justify-between px-6 py-4 border-b border-gray-100 sticky top-0 bg-white">
          <h2 class="text-lg font-bold text-gray-800">📄 Detail Transaksi</h2>
          <button @click="detailModal.show = false" class="text-gray-400 hover:text-gray-600 text-2xl leading-none">&times;</button>
        </div>

        <!-- Body -->
        <div v-if="detailModal.data" class="px-6 py-5">

          <!-- Info -->
          <div class="text-center mb-4 pb-4 border-b border-dashed border-gray-200">
            <p class="text-2xl mb-1">🛒</p>
            <p class="font-bold text-gray-800">MINI POS</p>
            <p class="text-xs text-gray-500">Receipt</p>
          </div>

          <div class="space-y-2 text-sm mb-4 pb-4 border-b border-dashed border-gray-200">
            <div class="flex justify-between">
              <span class="text-gray-500">Invoice:</span>
              <span class="font-mono font-semibold">{{ detailModal.data.invoice_number ?? `#TRX${detailModal.data.id}` }}</span>
            </div>
            <div class="flex justify-between">
              <span class="text-gray-500">Tanggal:</span>
              <span>{{ formatDate(detailModal.data.created_at) }}</span>
            </div>
            <div class="flex justify-between">
              <span class="text-gray-500">Kasir:</span>
              <span>{{ detailModal.data.user?.name ?? '-' }}</span>
            </div>
            <div class="flex justify-between">
              <span class="text-gray-500">Pembayaran:</span>
              <span class="uppercase font-medium">{{ detailModal.data.payment_method }}</span>
            </div>
          </div>

          <!-- Items -->
          <div class="mb-4 pb-4 border-b border-dashed border-gray-200">
            <p class="text-xs text-gray-500 mb-2 font-semibold uppercase">Items:</p>
            <div class="space-y-2">
              <div
                v-for="item in (detailModal.data.items ?? detailModal.data.transaction_items ?? [])"
                :key="item.id"
                class="flex justify-between items-start text-sm"
              >
                <div class="flex-1">
                  <p class="text-gray-800">{{ item.product?.name ?? item.product_name ?? 'Product' }}</p>
                  <p class="text-xs text-gray-500">
                    {{ item.quantity }} x {{ formatCurrency(item.price) }}
                  </p>
                </div>
                <p class="font-medium text-gray-800">{{ formatCurrency(item.subtotal ?? (item.price * item.quantity)) }}</p>
              </div>
            </div>
          </div>

          <!-- Total -->
          <div class="space-y-1 text-sm">
            <div class="flex justify-between">
              <span class="text-gray-500">Subtotal:</span>
              <span>{{ formatCurrency(detailModal.data.subtotal ?? detailModal.data.total) }}</span>
            </div>
            <div v-if="detailModal.data.tax" class="flex justify-between">
              <span class="text-gray-500">Pajak:</span>
              <span>{{ formatCurrency(detailModal.data.tax) }}</span>
            </div>
            <div class="flex justify-between font-bold text-lg pt-2 border-t border-gray-200">
              <span>TOTAL:</span>
              <span class="text-primary-600">{{ formatCurrency(detailModal.data.total) }}</span>
            </div>
            <div v-if="detailModal.data.paid_amount" class="flex justify-between text-sm text-gray-600">
              <span>Bayar:</span>
              <span>{{ formatCurrency(detailModal.data.paid_amount) }}</span>
            </div>
            <div v-if="detailModal.data.change_amount" class="flex justify-between text-sm text-gray-600">
              <span>Kembali:</span>
              <span>{{ formatCurrency(detailModal.data.change_amount) }}</span>
            </div>
          </div>

          <div class="text-center mt-6 pt-4 border-t border-dashed border-gray-200">
            <p class="text-xs text-gray-400">Terima kasih telah berbelanja! 🙏</p>
          </div>
        </div>

        <!-- Footer -->
        <div class="px-6 py-4 border-t border-gray-100 sticky bottom-0 bg-white">
          <button
            @click="detailModal.show = false"
            class="w-full bg-primary-600 hover:bg-primary-700 text-white py-2.5 rounded-lg font-medium transition"
          >
            Tutup
          </button>
        </div>
      </div>
    </div>

  </AppLayout>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import AppLayout from '@/components/AppLayout.vue'
import api from '@/services/api'

// ============ STATE ============
const transactions = ref([])
const loading = ref(true)
const search = ref('')
const filterPayment = ref('')

const alert = ref({ show: false, type: 'success', message: '' })
const detailModal = ref({ show: false, data: null })

// ============ COMPUTED ============
const filteredTransactions = computed(() => {
  return transactions.value.filter(trx => {
    const invoice = (trx.invoice_number ?? `#TRX${trx.id}`).toLowerCase()
    const kasir = (trx.user?.name ?? '').toLowerCase()
    const q = search.value.toLowerCase()

    const matchSearch = invoice.includes(q) || kasir.includes(q)
    const matchPayment = filterPayment.value === '' || trx.payment_method === filterPayment.value

    return matchSearch && matchPayment
  })
})

// ============ HELPERS ============
const formatCurrency = (val) => {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    minimumFractionDigits: 0,
  }).format(val ?? 0)
}

const formatDate = (date) => {
  if (!date) return '-'
  return new Date(date).toLocaleString('id-ID', {
    day: '2-digit',
    month: 'short',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  })
}

const showAlert = (type, message) => {
  alert.value = { show: true, type, message }
  setTimeout(() => { alert.value.show = false }, 3000)
}

// ============ FETCH ============
const fetchTransactions = async () => {
  loading.value = true
  try {
    const res = await api.get('/transactions')
    transactions.value = res.data.data ?? res.data
  } catch (e) {
    showAlert('error', 'Gagal memuat transaksi.')
  } finally {
    loading.value = false
  }
}

const viewDetail = async (trx) => {
  detailModal.value = { show: true, data: trx }
  // Refetch detail untuk dapat items lengkap
  try {
    const res = await api.get(`/transactions/${trx.id}`)
    detailModal.value.data = res.data.data ?? res.data
  } catch (e) {
    console.error('Gagal load detail')
  }
}

// ============ LIFECYCLE ============
onMounted(() => {
  fetchTransactions()
})
</script>
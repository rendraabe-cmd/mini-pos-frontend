<script setup>
import { ref, computed, onMounted } from 'vue'
import AppLayout from '@/components/AppLayout.vue'
import api from '@/services/api'

const products = ref([])
const categories = ref([])
const cart = ref([])
const loading = ref(true)
const selectedCategory = ref(null)
const searchQuery = ref('')

const paymentAmount = ref(0)
const paymentMethod = ref('cash')
const processing = ref(false)
const showReceipt = ref(false)
const lastTransaction = ref(null)

// ============ COMPUTED ============
const filteredProducts = computed(() => {
  let result = products.value

  if (selectedCategory.value) {
    result = result.filter(p => p.category_id === selectedCategory.value)
  }

  if (searchQuery.value) {
    result = result.filter(p =>
      p.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  }

  return result
})

const cartTotal = computed(() => {
  return cart.value.reduce((sum, item) => sum + (item.price * item.quantity), 0)
})

const cartItemCount = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.quantity, 0)
})

const changeAmount = computed(() => {
  return Math.max(0, paymentAmount.value - cartTotal.value)
})

const isPaymentValid = computed(() => {
  return paymentAmount.value >= cartTotal.value && cart.value.length > 0
})

// ============ METHODS ============
function formatRupiah(amount) {
  return 'Rp ' + Number(amount).toLocaleString('id-ID')
}

function addToCart(product) {
  const existing = cart.value.find(item => item.id === product.id)
  
  if (existing) {
    if (existing.quantity < product.stock) {
      existing.quantity++
    } else {
      alert(`Stok ${product.name} hanya tersisa ${product.stock}`)
    }
  } else {
    cart.value.push({
      id: product.id,
      name: product.name,
      price: Number(product.price),
      quantity: 1,
      stock: product.stock,
    })
  }
}

function increaseQty(item) {
  if (item.quantity < item.stock) {
    item.quantity++
  } else {
    alert(`Stok hanya tersisa ${item.stock}`)
  }
}

function decreaseQty(item) {
  if (item.quantity > 1) {
    item.quantity--
  } else {
    removeFromCart(item)
  }
}

function removeFromCart(item) {
  cart.value = cart.value.filter(i => i.id !== item.id)
}

function clearCart() {
  if (confirm('Hapus semua item dari cart?')) {
    cart.value = []
    paymentAmount.value = 0
  }
}

function setQuickPayment(amount) {
  paymentAmount.value = amount
}

async function checkout() {
  if (!isPaymentValid.value) return

  processing.value = true

  try {
    const payload = {
      items: cart.value.map(item => ({
        product_id: item.id,
        quantity: item.quantity,
      })),
      payment_amount: paymentAmount.value,
      payment_method: paymentMethod.value,
    }

    const response = await api.post('/transactions', payload)

    if (response.data.success) {
      lastTransaction.value = response.data.data
      showReceipt.value = true
      cart.value = []
      paymentAmount.value = 0
      await fetchProducts() // refresh stock
    }
  } catch (error) {
    alert('Transaksi gagal: ' + (error.response?.data?.message || error.message))
  } finally {
    processing.value = false
  }
}

function closeReceipt() {
  showReceipt.value = false
  lastTransaction.value = null
}

// ============ FETCH ============
async function fetchProducts() {
  try {
    const response = await api.get('/products')
    if (response.data.success) {
      products.value = response.data.data
    }
  } catch (error) {
    console.error('Failed to fetch products:', error)
  }
}

async function fetchCategories() {
  try {
    const response = await api.get('/categories')
    if (response.data.success) {
      categories.value = response.data.data
    }
  } catch (error) {
    console.error('Failed to fetch categories:', error)
  }
}

onMounted(async () => {
  loading.value = true
  await Promise.all([fetchProducts(), fetchCategories()])
  loading.value = false
})
</script>

<template>
  <AppLayout>
    <div class="h-screen flex flex-col">
      
      <!-- Header -->
      <div class="bg-white p-4 shadow-sm border-b border-gray-200">
        <h1 class="text-2xl font-bold text-gray-800">POS / Kasir 🛒</h1>
      </div>

      <!-- Main 2-column layout -->
      <div class="flex-1 flex overflow-hidden">
        
        <!-- LEFT: Products List -->
        <div class="flex-1 p-4 overflow-y-auto">
          
          <!-- Search & Filter -->
          <div class="mb-4 space-y-3">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="🔍 Cari produk..."
              class="w-full px-4 py-2.5 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary-500"
            />

            <div class="flex gap-2 overflow-x-auto pb-2">
              <button
                @click="selectedCategory = null"
                :class="!selectedCategory ? 'bg-primary-600 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'"
                class="px-4 py-2 rounded-lg text-sm font-medium whitespace-nowrap border border-gray-200 transition"
              >
                Semua
              </button>
              <button
                v-for="cat in categories"
                :key="cat.id"
                @click="selectedCategory = cat.id"
                :class="selectedCategory === cat.id ? 'bg-primary-600 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'"
                class="px-4 py-2 rounded-lg text-sm font-medium whitespace-nowrap border border-gray-200 transition"
              >
                {{ cat.name }}
              </button>
            </div>
          </div>

          <!-- Loading -->
          <div v-if="loading" class="text-center py-12 text-gray-500">
            Loading products...
          </div>

          <!-- Products Grid -->
          <div v-else class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3">
            <button
              v-for="product in filteredProducts"
              :key="product.id"
              @click="addToCart(product)"
              :disabled="product.stock === 0"
              class="bg-white p-4 rounded-xl shadow-sm hover:shadow-md hover:border-primary-300 border border-gray-200 transition text-left disabled:opacity-50 disabled:cursor-not-allowed"
            >
              <div class="text-3xl mb-2">📦</div>
              <p class="font-medium text-gray-800 text-sm line-clamp-2">{{ product.name }}</p>
              <p class="text-xs text-gray-500 mt-1">Stok: {{ product.stock }}</p>
              <p class="font-bold text-primary-600 mt-2">{{ formatRupiah(product.price) }}</p>
            </button>
          </div>

          <div v-if="!loading && filteredProducts.length === 0" class="text-center py-12 text-gray-400">
            Tidak ada produk ditemukan
          </div>
        </div>

        <!-- RIGHT: Cart -->
        <div class="w-96 bg-white border-l border-gray-200 flex flex-col">
          
          <!-- Cart Header -->
          <div class="p-4 border-b border-gray-200 flex justify-between items-center">
            <div>
              <h2 class="font-bold text-gray-800">Cart 🛍️</h2>
              <p class="text-xs text-gray-500">{{ cartItemCount }} items</p>
            </div>
            <button
              v-if="cart.length > 0"
              @click="clearCart"
              class="text-xs text-red-600 hover:text-red-800"
            >
              Clear All
            </button>
          </div>

          <!-- Cart Items -->
          <div class="flex-1 overflow-y-auto p-4 space-y-3">
            <div v-if="cart.length === 0" class="text-center py-12 text-gray-400 text-sm">
              Cart kosong<br>
              Klik produk untuk menambahkan
            </div>
            
            <div
              v-for="item in cart"
              :key="item.id"
              class="bg-gray-50 p-3 rounded-lg"
            >
              <div class="flex justify-between items-start mb-2">
                <div class="flex-1">
                  <p class="font-medium text-sm text-gray-800">{{ item.name }}</p>
                  <p class="text-xs text-gray-500">{{ formatRupiah(item.price) }}</p>
                </div>
                <button
                  @click="removeFromCart(item)"
                  class="text-red-500 hover:text-red-700 text-sm ml-2"
                >
                  ✕
                </button>
              </div>
              
              <div class="flex justify-between items-center">
                <div class="flex items-center gap-2">
                  <button
                    @click="decreaseQty(item)"
                    class="w-7 h-7 rounded bg-white border border-gray-300 hover:bg-gray-100"
                  >−</button>
                  <span class="font-medium text-sm w-6 text-center">{{ item.quantity }}</span>
                  <button
                    @click="increaseQty(item)"
                    class="w-7 h-7 rounded bg-white border border-gray-300 hover:bg-gray-100"
                  >+</button>
                </div>
                <p class="font-semibold text-sm text-primary-600">
                  {{ formatRupiah(item.price * item.quantity) }}
                </p>
              </div>
            </div>
          </div>

          <!-- Checkout Section -->
          <div v-if="cart.length > 0" class="border-t border-gray-200 p-4 space-y-3 bg-gray-50">
            
            <!-- Total -->
            <div class="flex justify-between items-center pb-2 border-b border-gray-300">
              <span class="font-bold text-gray-800">TOTAL</span>
              <span class="font-bold text-xl text-primary-700">{{ formatRupiah(cartTotal) }}</span>
            </div>

            <!-- Payment Method -->
            <div>
              <label class="text-xs font-medium text-gray-600 mb-1 block">Metode Pembayaran</label>
              <div class="flex gap-2">
                <button
                  @click="paymentMethod = 'cash'"
                  :class="paymentMethod === 'cash' ? 'bg-primary-600 text-white' : 'bg-white text-gray-700'"
                  class="flex-1 py-2 rounded-lg text-sm font-medium border border-gray-300"
                >💵 Cash</button>
                <button
                  @click="paymentMethod = 'qris'"
                  :class="paymentMethod === 'qris' ? 'bg-primary-600 text-white' : 'bg-white text-gray-700'"
                  class="flex-1 py-2 rounded-lg text-sm font-medium border border-gray-300"
                >📱 QRIS</button>
              </div>
            </div>

            <!-- Payment Amount -->
            <div>
              <label class="text-xs font-medium text-gray-600 mb-1 block">Jumlah Bayar</label>
              <input
                v-model.number="paymentAmount"
                type="number"
                placeholder="0"
                class="w-full px-3 py-2 border border-gray-300 rounded-lg text-right font-bold"
              />
              <div class="flex gap-1 mt-2">
                <button
                  v-for="amount in [50000, 100000, 200000]"
                  :key="amount"
                  @click="setQuickPayment(amount)"
                  class="flex-1 py-1 text-xs bg-white border border-gray-300 rounded hover:bg-gray-100"
                >{{ formatRupiah(amount) }}</button>
              </div>
            </div>

            <!-- Change -->
            <div class="flex justify-between text-sm">
              <span class="text-gray-600">Kembalian:</span>
              <span class="font-semibold text-green-600">{{ formatRupiah(changeAmount) }}</span>
            </div>

            <!-- Checkout Button -->
            <button
              @click="checkout"
              :disabled="!isPaymentValid || processing"
              class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-3 rounded-lg transition disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {{ processing ? 'Processing...' : '✅ BAYAR' }}
            </button>
          </div>
        </div>

      </div>

      <!-- ========== RECEIPT MODAL ========== -->
      <div
        v-if="showReceipt && lastTransaction"
        class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      >
        <div class="bg-white rounded-2xl max-w-md w-full p-6 max-h-[90vh] overflow-y-auto">
          <div class="text-center mb-4">
            <div class="text-5xl mb-2">✅</div>
            <h2 class="text-xl font-bold text-gray-800">Transaksi Berhasil!</h2>
            <p class="text-xs text-gray-500 mt-1">{{ lastTransaction.invoice_number }}</p>
          </div>

          <div class="border-t border-b border-dashed border-gray-300 py-3 my-3 space-y-2">
            <div v-for="item in lastTransaction.items" :key="item.id" class="flex justify-between text-sm">
              <div>
                <p class="font-medium">{{ item.product_name }}</p>
                <p class="text-xs text-gray-500">{{ item.quantity }} x {{ formatRupiah(item.price) }}</p>
              </div>
              <p class="font-semibold">{{ formatRupiah(item.subtotal) }}</p>
            </div>
          </div>

          <div class="space-y-2 text-sm">
            <div class="flex justify-between font-bold">
              <span>TOTAL</span>
              <span>{{ formatRupiah(lastTransaction.total_amount) }}</span>
            </div>
            <div class="flex justify-between text-gray-600">
              <span>Bayar ({{ lastTransaction.payment_method }})</span>
              <span>{{ formatRupiah(lastTransaction.payment_amount) }}</span>
            </div>
            <div class="flex justify-between text-green-600 font-semibold">
              <span>Kembalian</span>
              <span>{{ formatRupiah(lastTransaction.change_amount) }}</span>
            </div>
          </div>

          <button
            @click="closeReceipt"
            class="w-full mt-6 bg-primary-600 hover:bg-primary-700 text-white font-bold py-3 rounded-lg"
          >
            Tutup
          </button>
        </div>
      </div>

    </div>
  </AppLayout>
</template>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
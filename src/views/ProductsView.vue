<template>
  <AppLayout>
    <div class="p-6">

      <!-- Header -->
      <div class="flex items-center justify-between mb-6">
        <div>
          <h1 class="text-2xl font-bold text-gray-800">Products</h1>
          <p class="text-gray-500 text-sm mt-1">Kelola produk yang dijual</p>
        </div>
        <button
          @click="openAddModal"
          class="bg-primary-600 hover:bg-primary-700 text-white px-4 py-2 rounded-lg font-medium flex items-center gap-2 transition"
        >
          <span class="text-lg">+</span> Tambah Produk
        </button>
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

      <!-- Filter & Search -->
      <div class="flex gap-3 mb-4">
        <input
          v-model="search"
          type="text"
          placeholder="🔍 Cari produk..."
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500 w-64"
        />
        <select
          v-model="filterCategory"
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500"
        >
          <option value="">Semua Kategori</option>
          <option v-for="cat in categories" :key="cat.id" :value="cat.id">
            {{ cat.name }}
          </option>
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
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">No</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Produk</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Kategori</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Harga</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Stok</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Status</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Aksi</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-50">
            <tr v-if="filteredProducts.length === 0">
              <td colspan="7" class="text-center py-12 text-gray-400">
                <div class="text-4xl mb-2">📦</div>
                <p>Belum ada produk</p>
              </td>
            </tr>
            <tr
              v-for="(product, index) in filteredProducts"
              :key="product.id"
              class="hover:bg-gray-50 transition"
            >
              <td class="px-6 py-4 text-sm text-gray-500">{{ index + 1 }}</td>
              <td class="px-6 py-4">
                <div class="flex items-center gap-3">
                  <div class="w-10 h-10 rounded-lg bg-primary-100 flex items-center justify-center text-xl">
                    📦
                  </div>
                  <div>
                    <p class="font-medium text-gray-800">{{ product.name }}</p>
                    <p class="text-xs text-gray-400">SKU: #{{ product.id }}</p>
                  </div>
                </div>
              </td>
              <td class="px-6 py-4">
                <span class="bg-blue-50 text-blue-700 text-xs px-2 py-1 rounded-full font-medium">
                  {{ product.category?.name ?? '-' }}
                </span>
              </td>
              <td class="px-6 py-4 text-sm font-semibold text-gray-800">
                {{ formatCurrency(product.price) }}
              </td>
              <td class="px-6 py-4">
                <span :class="[
                  'text-sm font-medium',
                  product.stock === 0 ? 'text-red-600' :
                  product.stock < 10 ? 'text-yellow-600' : 'text-green-600'
                ]">
                  {{ product.stock }}
                </span>
              </td>
              <td class="px-6 py-4">
                <span :class="[
                  'text-xs px-2 py-1 rounded-full font-medium',
                  product.is_active
                    ? 'bg-green-100 text-green-700'
                    : 'bg-gray-100 text-gray-500'
                ]">
                  {{ product.is_active ? 'Aktif' : 'Non-aktif' }}
                </span>
              </td>
              <td class="px-6 py-4">
                <div class="flex gap-2">
                  <button
                    @click="openEditModal(product)"
                    class="bg-yellow-50 hover:bg-yellow-100 text-yellow-700 px-3 py-1.5 rounded-lg text-sm font-medium transition"
                  >
                    ✏️ Edit
                  </button>
                  <button
                    @click="confirmDelete(product)"
                    class="bg-red-50 hover:bg-red-100 text-red-700 px-3 py-1.5 rounded-lg text-sm font-medium transition"
                  >
                    🗑️ Hapus
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

    </div>

    <!-- Modal Add/Edit -->
    <div v-if="modal.show" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
      <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-y-auto">

        <!-- Modal Header -->
        <div class="flex items-center justify-between px-6 py-4 border-b border-gray-100 sticky top-0 bg-white">
          <h2 class="text-lg font-bold text-gray-800">
            {{ modal.mode === 'add' ? '➕ Tambah Produk' : '✏️ Edit Produk' }}
          </h2>
          <button @click="closeModal" class="text-gray-400 hover:text-gray-600 text-2xl leading-none">&times;</button>
        </div>

        <!-- Modal Body -->
        <div class="px-6 py-5 space-y-4">

          <!-- Nama -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Nama Produk <span class="text-red-500">*</span>
            </label>
            <input
              v-model="form.name"
              type="text"
              placeholder="Contoh: Nasi Goreng Spesial"
              class="w-full border border-gray-200 rounded-lg px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500"
            />
          </div>

          <!-- Kategori -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Kategori <span class="text-red-500">*</span>
            </label>
            <select
              v-model="form.category_id"
              class="w-full border border-gray-200 rounded-lg px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500"
            >
              <option value="">-- Pilih Kategori --</option>
              <option v-for="cat in categories" :key="cat.id" :value="cat.id">
                {{ cat.name }}
              </option>
            </select>
          </div>

          <!-- Harga & Stok -->
          <div class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">
                Harga <span class="text-red-500">*</span>
              </label>
              <div class="relative">
                <span class="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-sm">Rp</span>
                <input
                  v-model="form.price"
                  type="number"
                  min="0"
                  placeholder="0"
                  class="w-full border border-gray-200 rounded-lg pl-9 pr-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500"
                />
              </div>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">
                Stok <span class="text-red-500">*</span>
              </label>
              <input
                v-model="form.stock"
                type="number"
                min="0"
                placeholder="0"
                class="w-full border border-gray-200 rounded-lg px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500"
              />
            </div>
          </div>

          <!-- Deskripsi -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Deskripsi</label>
            <textarea
              v-model="form.description"
              rows="3"
              placeholder="Deskripsi produk (opsional)..."
              class="w-full border border-gray-200 rounded-lg px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500 resize-none"
            ></textarea>
          </div>

          <!-- Status Aktif -->
          <div class="flex items-center gap-3">
            <button
              @click="form.is_active = !form.is_active"
              :class="[
                'relative w-12 h-6 rounded-full transition-colors duration-200',
                form.is_active ? 'bg-primary-600' : 'bg-gray-300'
              ]"
            >
              <span :class="[
                'absolute top-1 w-4 h-4 bg-white rounded-full shadow transition-transform duration-200',
                form.is_active ? 'translate-x-7' : 'translate-x-1'
              ]"></span>
            </button>
            <span class="text-sm text-gray-700">
              {{ form.is_active ? '✅ Produk Aktif' : '⭕ Produk Non-aktif' }}
            </span>
          </div>

          <!-- Error -->
          <p v-if="formError" class="text-red-500 text-sm bg-red-50 px-3 py-2 rounded-lg">
            {{ formError }}
          </p>

        </div>

        <!-- Modal Footer -->
        <div class="flex gap-3 px-6 py-4 border-t border-gray-100 sticky bottom-0 bg-white">
          <button
            @click="closeModal"
            class="flex-1 border border-gray-200 text-gray-700 py-2.5 rounded-lg font-medium hover:bg-gray-50 transition"
          >
            Batal
          </button>
          <button
            @click="submitForm"
            :disabled="submitting"
            class="flex-1 bg-primary-600 hover:bg-primary-700 text-white py-2.5 rounded-lg font-medium transition disabled:opacity-50"
          >
            {{ submitting ? 'Menyimpan...' : (modal.mode === 'add' ? 'Tambah' : 'Simpan') }}
          </button>
        </div>

      </div>
    </div>

    <!-- Modal Konfirmasi Hapus -->
    <div v-if="deleteModal.show" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
      <div class="bg-white rounded-2xl shadow-2xl w-full max-w-sm">
        <div class="px-6 py-6 text-center">
          <div class="text-5xl mb-3">🗑️</div>
          <h3 class="text-lg font-bold text-gray-800 mb-1">Hapus Produk?</h3>
          <p class="text-gray-500 text-sm mb-6">
            Yakin ingin menghapus <strong>"{{ deleteModal.product?.name }}"</strong>?
            Tindakan ini tidak bisa dibatalkan.
          </p>
          <div class="flex gap-3">
            <button
              @click="deleteModal.show = false"
              class="flex-1 border border-gray-200 text-gray-700 py-2.5 rounded-lg font-medium hover:bg-gray-50 transition"
            >
              Batal
            </button>
            <button
              @click="deleteProduct"
              :disabled="submitting"
              class="flex-1 bg-red-600 hover:bg-red-700 text-white py-2.5 rounded-lg font-medium transition disabled:opacity-50"
            >
              {{ submitting ? 'Menghapus...' : 'Hapus' }}
            </button>
          </div>
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
const products = ref([])
const categories = ref([])
const loading = ref(true)
const submitting = ref(false)
const search = ref('')
const filterCategory = ref('')

const alert = ref({ show: false, type: 'success', message: '' })
const modal = ref({ show: false, mode: 'add' })
const deleteModal = ref({ show: false, product: null })

const form = ref({
  name: '',
  category_id: '',
  price: '',
  stock: '',
  description: '',
  is_active: true,
})
const formError = ref('')
const editingId = ref(null)

// ============ COMPUTED ============
const filteredProducts = computed(() => {
  return products.value.filter(p => {
    const matchSearch = p.name.toLowerCase().includes(search.value.toLowerCase())
    const matchCategory = filterCategory.value === '' || p.category_id == filterCategory.value
    return matchSearch && matchCategory
  })
})

// ============ HELPERS ============
const formatCurrency = (val) => {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    minimumFractionDigits: 0,
  }).format(val)
}

const showAlert = (type, message) => {
  alert.value = { show: true, type, message }
  setTimeout(() => { alert.value.show = false }, 3000)
}

// ============ FETCH ============
const fetchProducts = async () => {
  loading.value = true
  try {
    const res = await api.get('/products')
    products.value = res.data.data ?? res.data
  } catch (e) {
    showAlert('error', 'Gagal memuat produk.')
  } finally {
    loading.value = false
  }
}

const fetchCategories = async () => {
  try {
    const res = await api.get('/categories')
    categories.value = res.data.data ?? res.data
  } catch (e) {
    console.error('Gagal load kategori')
  }
}

// ============ MODAL ============
const resetForm = () => {
  form.value = {
    name: '',
    category_id: '',
    price: '',
    stock: '',
    description: '',
    is_active: true,
  }
  formError.value = ''
  editingId.value = null
}

const openAddModal = () => {
  resetForm()
  modal.value = { show: true, mode: 'add' }
}

const openEditModal = (product) => {
  form.value = {
    name: product.name,
    category_id: product.category_id,
    price: product.price,
    stock: product.stock,
    description: product.description ?? '',
    is_active: product.is_active,
  }
  formError.value = ''
  editingId.value = product.id
  modal.value = { show: true, mode: 'edit' }
}

const closeModal = () => {
  modal.value.show = false
}

// ============ SUBMIT ============
const submitForm = async () => {
  // Validasi
  if (!form.value.name.trim()) {
    formError.value = 'Nama produk wajib diisi.'
    return
  }
  if (!form.value.category_id) {
    formError.value = 'Kategori wajib dipilih.'
    return
  }
  if (form.value.price === '' || form.value.price < 0) {
    formError.value = 'Harga tidak valid.'
    return
  }
  if (form.value.stock === '' || form.value.stock < 0) {
    formError.value = 'Stok tidak valid.'
    return
  }

  formError.value = ''
  submitting.value = true

  const payload = {
    name: form.value.name.trim(),
    category_id: form.value.category_id,
    price: Number(form.value.price),
    stock: Number(form.value.stock),
    description: form.value.description,
    is_active: form.value.is_active,
  }

  try {
    if (modal.value.mode === 'add') {
      await api.post('/products', payload)
      showAlert('success', '✅ Produk berhasil ditambahkan!')
    } else {
      await api.put(`/products/${editingId.value}`, payload)
      showAlert('success', '✅ Produk berhasil diperbarui!')
    }
    closeModal()
    fetchProducts()
  } catch (e) {
    const msg = e.response?.data?.message ?? 'Terjadi kesalahan.'
    formError.value = `❌ ${msg}`
  } finally {
    submitting.value = false
  }
}

// ============ DELETE ============
const confirmDelete = (product) => {
  deleteModal.value = { show: true, product }
}

const deleteProduct = async () => {
  submitting.value = true
  try {
    await api.delete(`/products/${deleteModal.value.product.id}`)
    showAlert('success', '✅ Produk berhasil dihapus!')
    deleteModal.value.show = false
    fetchProducts()
  } catch (e) {
    const msg = e.response?.data?.message ?? 'Gagal menghapus produk.'
    showAlert('error', `❌ ${msg}`)
    deleteModal.value.show = false
  } finally {
    submitting.value = false
  }
}

// ============ LIFECYCLE ============
onMounted(() => {
  fetchProducts()
  fetchCategories()
})
</script>
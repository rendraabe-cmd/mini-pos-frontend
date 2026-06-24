<template>
  <AppLayout>
    <div class="p-6">

      <!-- Header -->
      <div class="flex items-center justify-between mb-6">
        <div>
          <h1 class="text-2xl font-bold text-gray-800">Categories</h1>
          <p class="text-gray-500 text-sm mt-1">Kelola kategori produk</p>
        </div>
        <button
          @click="openAddModal"
          class="bg-primary-600 hover:bg-primary-700 text-white px-4 py-2 rounded-lg font-medium flex items-center gap-2 transition"
        >
          <span class="text-lg">+</span> Tambah Kategori
        </button>
      </div>

      <!-- Alert -->
      <div v-if="alert.show" :class="[
        'mb-4 px-4 py-3 rounded-lg text-sm font-medium',
        alert.type === 'success' ? 'bg-green-100 text-green-800 border border-green-200' : 'bg-red-100 text-red-800 border border-red-200'
      ]">
        {{ alert.message }}
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
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Nama Kategori</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Slug</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Jumlah Produk</th>
              <th class="text-left px-6 py-4 text-xs font-semibold text-gray-500 uppercase tracking-wider">Aksi</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-50">
            <tr v-if="categories.length === 0">
              <td colspan="5" class="text-center py-12 text-gray-400">
                <div class="text-4xl mb-2">📂</div>
                <p>Belum ada kategori</p>
              </td>
            </tr>
            <tr
              v-for="(cat, index) in categories"
              :key="cat.id"
              class="hover:bg-gray-50 transition"
            >
              <td class="px-6 py-4 text-sm text-gray-500">{{ index + 1 }}</td>
              <td class="px-6 py-4">
                <div class="flex items-center gap-3">
                  <div class="w-8 h-8 rounded-lg bg-primary-100 flex items-center justify-center text-primary-600 font-bold text-sm">
                    {{ cat.name.charAt(0).toUpperCase() }}
                  </div>
                  <span class="font-medium text-gray-800">{{ cat.name }}</span>
                </div>
              </td>
              <td class="px-6 py-4 text-sm text-gray-500">
                <span class="bg-gray-100 px-2 py-1 rounded text-xs font-mono">{{ cat.slug }}</span>
              </td>
              <td class="px-6 py-4 text-sm text-gray-600">
                {{ cat.products_count ?? cat.products?.length ?? 0 }} produk
              </td>
              <td class="px-6 py-4">
                <div class="flex gap-2">
                  <button
                    @click="openEditModal(cat)"
                    class="bg-yellow-50 hover:bg-yellow-100 text-yellow-700 px-3 py-1.5 rounded-lg text-sm font-medium transition"
                  >
                    ✏️ Edit
                  </button>
                  <button
                    @click="confirmDelete(cat)"
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
      <div class="bg-white rounded-2xl shadow-2xl w-full max-w-md">

        <!-- Modal Header -->
        <div class="flex items-center justify-between px-6 py-4 border-b border-gray-100">
          <h2 class="text-lg font-bold text-gray-800">
            {{ modal.mode === 'add' ? '➕ Tambah Kategori' : '✏️ Edit Kategori' }}
          </h2>
          <button @click="closeModal" class="text-gray-400 hover:text-gray-600 text-2xl leading-none">&times;</button>
        </div>

        <!-- Modal Body -->
        <div class="px-6 py-5">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Nama Kategori <span class="text-red-500">*</span>
            </label>
            <input
              v-model="form.name"
              type="text"
              placeholder="Contoh: Makanan, Minuman..."
              class="w-full border border-gray-200 rounded-lg px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent"
              @keyup.enter="submitForm"
            />
            <p v-if="formError" class="text-red-500 text-xs mt-1">{{ formError }}</p>
          </div>

          <!-- Preview Slug -->
          <div v-if="form.name" class="bg-gray-50 rounded-lg px-3 py-2 text-xs text-gray-500">
            Slug: <span class="font-mono text-gray-700">{{ slugPreview }}</span>
          </div>
        </div>

        <!-- Modal Footer -->
        <div class="flex gap-3 px-6 py-4 border-t border-gray-100">
          <button
            @click="closeModal"
            class="flex-1 border border-gray-200 text-gray-700 py-2.5 rounded-lg font-medium hover:bg-gray-50 transition"
          >
            Batal
          </button>
          <button
            @click="submitForm"
            :disabled="submitting"
            class="flex-1 bg-primary-600 hover:bg-primary-700 text-white py-2.5 rounded-lg font-medium transition disabled:opacity-50 disabled:cursor-not-allowed"
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
          <h3 class="text-lg font-bold text-gray-800 mb-1">Hapus Kategori?</h3>
          <p class="text-gray-500 text-sm mb-6">
            Yakin ingin menghapus <strong>"{{ deleteModal.category?.name }}"</strong>?
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
              @click="deleteCategory"
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
const categories = ref([])
const loading = ref(true)
const submitting = ref(false)

const alert = ref({ show: false, type: 'success', message: '' })

const modal = ref({ show: false, mode: 'add' })
const deleteModal = ref({ show: false, category: null })

const form = ref({ name: '' })
const formError = ref('')
const editingId = ref(null)

// ============ COMPUTED ============
const slugPreview = computed(() => {
  return form.value.name
    .toLowerCase()
    .replace(/[^a-z0-9\s-]/g, '')
    .replace(/\s+/g, '-')
    .replace(/-+/g, '-')
    .trim()
})

// ============ METHODS ============
const showAlert = (type, message) => {
  alert.value = { show: true, type, message }
  setTimeout(() => { alert.value.show = false }, 3000)
}

const fetchCategories = async () => {
  loading.value = true
  try {
    const res = await api.get('/categories')
    categories.value = res.data.data ?? res.data
  } catch (e) {
    showAlert('error', 'Gagal memuat kategori.')
  } finally {
    loading.value = false
  }
}

const openAddModal = () => {
  form.value = { name: '' }
  formError.value = ''
  editingId.value = null
  modal.value = { show: true, mode: 'add' }
}

const openEditModal = (cat) => {
  form.value = { name: cat.name }
  formError.value = ''
  editingId.value = cat.id
  modal.value = { show: true, mode: 'edit' }
}

const closeModal = () => {
  modal.value.show = false
}

const submitForm = async () => {
  // Validasi
  if (!form.value.name.trim()) {
    formError.value = 'Nama kategori wajib diisi.'
    return
  }
  formError.value = ''
  submitting.value = true

  try {
    if (modal.value.mode === 'add') {
      await api.post('/categories', { name: form.value.name.trim() })
      showAlert('success', '✅ Kategori berhasil ditambahkan!')
    } else {
      await api.put(`/categories/${editingId.value}`, { name: form.value.name.trim() })
      showAlert('success', '✅ Kategori berhasil diperbarui!')
    }
    closeModal()
    fetchCategories()
  } catch (e) {
    const msg = e.response?.data?.message ?? 'Terjadi kesalahan.'
    showAlert('error', `❌ ${msg}`)
  } finally {
    submitting.value = false
  }
}

const confirmDelete = (cat) => {
  deleteModal.value = { show: true, category: cat }
}

const deleteCategory = async () => {
  submitting.value = true
  try {
    await api.delete(`/categories/${deleteModal.value.category.id}`)
    showAlert('success', '✅ Kategori berhasil dihapus!')
    deleteModal.value.show = false
    fetchCategories()
  } catch (e) {
    const msg = e.response?.data?.message ?? 'Gagal menghapus kategori.'
    showAlert('error', `❌ ${msg}`)
    deleteModal.value.show = false
  } finally {
    submitting.value = false
  }
}

// ============ LIFECYCLE ============
onMounted(() => {
  fetchCategories()
})
</script>
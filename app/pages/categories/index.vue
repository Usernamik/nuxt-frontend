<template>
  <div class="p-6 max-w-6xl mx-auto space-y-6">
    <!-- Header -->
    <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
      <div>
        <h1 class="text-3xl font-bold tracking-tight">Категорії</h1>
        <p class="text-sm text-muted mt-1">Керування категоріями блогу</p>
      </div>
      <UButton to="/categories/create" color="primary" size="lg" class="shrink-0">
        <template #leading>
          <Icon name="i-lucide-plus" class="w-4 h-4" />
        </template>
        Додати категорію
      </UButton>
    </div>

    <!-- Search -->
    <UInput
      v-model="searchQuery"
      placeholder="Пошук категорій..."
      class="w-full sm:max-w-xs"
      size="md"
    >
      <template #leading>
        <Icon name="i-lucide-search" class="w-4 h-4 text-muted" />
      </template>
    </UInput>

    <!-- Loading State -->
    <template v-if="loading && categories.length === 0">
      <div class="space-y-3">
        <div v-for="n in 4" :key="n" class="flex items-center gap-4 p-4 rounded-xl bg-muted/50 animate-pulse">
          <div class="w-10 h-10 rounded-xl bg-muted" />
          <div class="flex-1 space-y-2">
            <div class="h-4 w-1/3 rounded bg-muted" />
            <div class="h-3 w-1/4 rounded bg-muted" />
          </div>
          <div class="h-8 w-20 rounded-lg bg-muted" />
        </div>
      </div>
    </template>

    <!-- Empty State -->
    <template v-else-if="filteredCategories.length === 0 && !loading">
      <div class="flex flex-col items-center justify-center py-20 text-center">
        <div class="w-16 h-16 rounded-2xl bg-muted flex items-center justify-center mb-4">
          <Icon name="i-lucide-folder" class="w-8 h-8 text-muted" />
        </div>
        <h3 class="text-lg font-semibold">Категорій не знайдено</h3>
        <p class="text-sm text-muted mt-1 mb-6">
          {{ searchQuery ? 'Спробуйте змінити параметри пошуку' : 'Створіть свою першу категорію' }}
        </p>
        <UButton v-if="!searchQuery" to="/categories/create" color="primary">
          <template #leading>
            <Icon name="i-lucide-plus" class="w-4 h-4" />
          </template>
          Додати категорію
        </UButton>
      </div>
    </template>

    <!-- Table -->
    <template v-else>
      <UCard class="overflow-hidden" variant="outline">
        <UTable :data="filteredCategories" :columns="columns" class="w-full">
          <template #title-data="{ row }">
            <div class="flex items-center gap-3">
              <div class="w-9 h-9 rounded-xl bg-primary/10 flex items-center justify-center">
                <Icon name="i-lucide-folder" class="w-4 h-4 text-primary" />
              </div>
              <div>
                <span class="font-medium">{{ row.title }}</span>
                <span v-if="row.slug" class="text-xs text-muted ml-2">/{{ row.slug }}</span>
              </div>
            </div>
          </template>

          <template #slug-data="{ row }">
            <span class="text-sm font-mono text-muted">
              {{ row.slug || '—' }}
            </span>
          </template>

          <template #parent_title-data="{ row }">
            <span class="text-sm text-muted">
              {{ row.parent_title || '—' }}
            </span>
          </template>

          <template #actions-data="{ row }">
            <div class="flex items-center gap-1">
              <UTooltip text="Редагувати">
                <UButton
                  :to="`/categories/${row.id}/edit`"
                  icon="i-lucide-pencil"
                  color="neutral"
                  variant="ghost"
                  size="sm"
                />
              </UTooltip>
              <UDropdownMenu :items="getRowItems(row)">
                <UButton icon="i-lucide-ellipsis-vertical" color="neutral" variant="ghost" size="sm" />
              </UDropdownMenu>
            </div>
          </template>
        </UTable>

        <!-- Pagination Footer -->
        <template #footer>
          <div class="flex items-center justify-between px-3 py-2">
            <span class="text-sm text-muted">
              {{ paginationMeta.from }}–{{ paginationMeta.to }} з {{ paginationMeta.total }}
            </span>
            <div class="flex items-center gap-1">
              <UButton
                color="neutral"
                variant="ghost"
                size="sm"
                :disabled="paginationMeta.current_page <= 1 || loading"
                @click="goToPage(paginationMeta.current_page - 1)"
              >
                <template #leading>
                  <Icon name="i-lucide-chevron-left" class="w-4 h-4" />
                </template>
              </UButton>

              <template v-for="link in paginationMeta.links" :key="link.label">
                <UButton
                  v-if="link.page && /^\d+$/.test(link.label) && Math.abs(link.page - paginationMeta.current_page) < 3"
                  color="neutral"
                  :variant="link.active ? 'solid' : 'ghost'"
                  size="sm"
                  class="min-w-[32px]"
                  @click="goToPage(link.page)"
                >
                  {{ link.label }}
                </UButton>
              </template>

              <UButton
                color="neutral"
                variant="ghost"
                size="sm"
                :disabled="paginationMeta.current_page >= paginationMeta.last_page || loading"
                @click="goToPage(paginationMeta.current_page + 1)"
              >
                <template #leading>
                  <Icon name="i-lucide-chevron-right" class="w-4 h-4" />
                </template>
              </UButton>
            </div>
          </div>
        </template>
      </UCard>
    </template>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const categories = ref([])
const loading = ref(false)
const searchQuery = ref('')
const toast = useToast()

const paginationMeta = ref({
  current_page: 1,
  last_page: 1,
  from: 0,
  to: 0,
  total: 0,
  links: []
})

const columns = [
  { accessorKey: 'id', header: '#' },
  { accessorKey: 'title', header: 'Назва' },
  { accessorKey: 'slug', header: 'Slug' },
  { accessorKey: 'parent_title', header: 'Батьківська' },
  { id: 'actions', header: 'Дії', enableSorting: false }
]

const filteredCategories = computed(() => {
  if (!searchQuery.value) return categories.value
  const q = searchQuery.value.toLowerCase()
  return categories.value.filter(c =>
    c.title?.toLowerCase().includes(q) || c.slug?.toLowerCase().includes(q)
  )
})

const fetchCategories = async (page = 1) => {
  loading.value = true
  try {
    const res = await fetch(`/api/admin/blog/categories?page=${page}`, { cache: 'no-store' })
    if (!res.ok) {
      console.error('[Categories List] Server responded:', res.status)
      throw new Error('Помилка сервера: ' + res.status)
    }
    const data = await res.json()
    const items = data.data || data

    if (data.meta) {
      paginationMeta.value = data.meta
    }

    categories.value = items
  } catch (err) {
    console.error('[Categories List] Error:', err)
    toast.add({ title: 'Помилка завантаження', description: err.message, color: 'error' })
  } finally {
    loading.value = false
  }
}

const goToPage = (page) => {
  if (page < 1 || page > paginationMeta.value.last_page || loading.value) return
  fetchCategories(page)
}

const deleteCategory = async (id) => {
  try {
    await fetch(`/api/admin/blog/categories/${id}`, { method: 'DELETE' })
    toast.add({ title: 'Категорію видалено', color: 'success' })
    fetchCategories(paginationMeta.value.current_page)
  } catch (err) {
    toast.add({ title: 'Помилка видалення', color: 'error' })
  }
}

const getRowItems = (row) => [
  { type: 'label', label: row.title },
  { type: 'separator' },
  { label: 'Редагувати', to: `/categories/${row.id}/edit` },
  { type: 'separator' },
  { label: 'Видалити', icon: 'i-lucide-trash-2', color: 'error', onSelect: () => deleteCategory(row.id) }
]

onMounted(() => {
  fetchCategories()
})

watch(() => useRoute().fullPath, () => {
  fetchCategories(paginationMeta.value.current_page)
})
</script>

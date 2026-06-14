<template>
  <div class="p-6 max-w-6xl mx-auto space-y-6">
    <!-- Header -->
    <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
      <div>
        <h1 class="text-3xl font-bold tracking-tight">Пости блогу</h1>
        <p class="text-sm text-muted mt-1">Керування публікаціями блогу</p>
      </div>
      <UButton to="/blog-posts/create" color="primary" size="lg" class="shrink-0">
        <template #leading>
          <Icon name="i-lucide-plus" class="w-4 h-4" />
        </template>
        Додати пост
      </UButton>
    </div>

    <!-- Search & Filter Bar -->
    <div class="flex flex-col sm:flex-row gap-3">
      <UInput
        v-model="searchQuery"
        placeholder="Пошук за заголовком..."
        class="w-full sm:max-w-xs"
        size="md"
      >
        <template #leading>
          <Icon name="i-lucide-search" class="w-4 h-4 text-muted" />
        </template>
      </UInput>
      <div class="flex gap-2 ml-auto">
        <USelect
          v-model="statusFilter"
          :items="statusOptions"
          size="md"
          class="w-40"
        />
        <UButton
          color="neutral"
          variant="outline"
          size="md"
          @click="refreshPosts"
          :disabled="loading"
        >
          <template #leading>
            <Icon name="i-lucide-refresh-cw" class="w-4 h-4" :class="{ 'animate-spin': loading }" />
          </template>
        </UButton>
      </div>
    </div>

    <!-- Loading State -->
    <template v-if="loading && posts.length === 0">
      <div class="space-y-3">
        <div v-for="n in 5" :key="n" class="flex items-center gap-4 p-4 rounded-xl bg-muted/50 animate-pulse">
          <div class="w-8 h-8 rounded-full bg-muted" />
          <div class="flex-1 space-y-2">
            <div class="h-4 w-3/4 rounded bg-muted" />
            <div class="h-3 w-1/2 rounded bg-muted" />
          </div>
          <div class="h-8 w-20 rounded-lg bg-muted" />
        </div>
      </div>
    </template>

    <!-- Empty State -->
    <template v-else-if="filteredPosts.length === 0 && !loading">
      <div class="flex flex-col items-center justify-center py-20 text-center">
        <div class="w-16 h-16 rounded-2xl bg-muted flex items-center justify-center mb-4">
          <Icon name="i-lucide-file-text" class="w-8 h-8 text-muted" />
        </div>
        <h3 class="text-lg font-semibold">Постів не знайдено</h3>
        <p class="text-sm text-muted mt-1 mb-6">
          {{ searchQuery ? 'Спробуйте змінити параметри пошуку' : 'Створіть свій перший пост' }}
        </p>
        <UButton v-if="!searchQuery" to="/blog-posts/create" color="primary">
          <template #leading>
            <Icon name="i-lucide-plus" class="w-4 h-4" />
          </template>
          Додати пост
        </UButton>
      </div>
    </template>

    <!-- Table -->
    <template v-else>
      <UCard class="overflow-x-auto" variant="outline">
        <UTable
          :data="filteredPosts"
          :columns="columns"
          class="w-full"
        >
          <!-- Author Column -->
          <template #user_id-data="{ row }">
            <div class="flex items-center gap-2">
              <div class="w-7 h-7 rounded-full bg-primary/10 flex items-center justify-center shrink-0">
                <Icon name="i-lucide-user" class="w-3.5 h-3.5 text-primary" />
              </div>
              <span class="text-sm whitespace-nowrap font-mono">
                #{{ row.user_id }}
              </span>
            </div>
          </template>

          <!-- Category Column -->
          <template #category_id-data="{ row }">
            <UBadge color="info" variant="subtle" size="sm">
              #{{ row.category_id }}
            </UBadge>
          </template>

          <!-- Status Column -->
          <template #is_published-data="{ row }">
            <UBadge
              :color="row.is_published ? 'success' : 'neutral'"
              :variant="row.is_published ? 'subtle' : 'outline'"
              size="sm"
              class="capitalize"
            >
              {{ row.is_published ? 'Опубліковано' : 'Чернетка' }}
            </UBadge>
          </template>

          <!-- Date Column -->
          <template #date_published-data="{ row }">
            <span class="text-sm text-muted whitespace-nowrap">
              {{ formatDate(row.date_published) }}
            </span>
          </template>

          <!-- Actions Column -->
          <template #actions-data="{ row }">
            <div class="flex items-center gap-1">
              <UTooltip text="Переглянути">
                <UButton
                  :to="`/blog-posts/${row.id}`"
                  icon="i-lucide-eye"
                  color="neutral"
                  variant="ghost"
                  size="sm"
                />
              </UTooltip>
              <UTooltip text="Редагувати">
                <UButton
                  :to="`/blog-posts/${row.id}/edit`"
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
            <div class="flex items-center gap-2 text-sm text-muted">
              <span>{{ paginationMeta.from }}–{{ paginationMeta.to }} з {{ paginationMeta.total }}</span>
              <span class="text-xs opacity-60">{{ filteredPosts.length }} на сторінці</span>
            </div>
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

const posts = ref([])
const loading = ref(false)
const searchQuery = ref('')
const statusFilter = ref('all')

const toast = useToast()

const paginationMeta = ref({
  current_page: 1,
  last_page: 1,
  from: 0,
  to: 0,
  total: 0,
  links: []
})

const statusOptions = [
  { label: 'Всі статуси', value: 'all' },
  { label: 'Опубліковані', value: 'published' },
  { label: 'Чернетки', value: 'draft' }
]

const columns = [
  { accessorKey: 'id', header: '#' },
  { accessorKey: 'user_id', header: 'Автор' },
  { accessorKey: 'category_id', header: 'Категорія' },
  { accessorKey: 'title', header: 'Заголовок' },
  { accessorKey: 'is_published', header: 'Статус' },
  { accessorKey: 'date_published', header: 'Дата публікації' },
  { id: 'actions', header: 'Дії' }
]


const filteredPosts = computed(() => {
  let result = posts.value

  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase()
    result = result.filter(p => p.title?.toLowerCase().includes(q))
  }

  if (statusFilter.value === 'published') {
    result = result.filter(p => p.is_published)
  } else if (statusFilter.value === 'draft') {
    result = result.filter(p => !p.is_published)
  }

  return result
})

const fetchPosts = async (page = 1) => {
  loading.value = true
  try {
    const res = await fetch(`/api/admin/blog/posts?page=${page}`, { cache: 'no-store' })
    const data = await res.json()
    const items = data.data || data

    if (data.meta) {
      paginationMeta.value = data.meta
    }

    posts.value = items
  } catch (err) {
    toast.add({ title: 'Помилка завантаження', color: 'error' })
  } finally {
    loading.value = false
  }
}

const goToPage = (page) => {
  if (page < 1 || page > paginationMeta.value.last_page || loading.value) return
  fetchPosts(page)
}

const refreshPosts = () => {
  fetchPosts(paginationMeta.value.current_page)
}

const deletePost = async (id) => {
  try {
    await fetch(`/api/admin/blog/posts/${id}`, { method: 'DELETE' })
    toast.add({ title: 'Пост видалено', color: 'success' })
    fetchPosts(paginationMeta.value.current_page)
  } catch (err) {
    toast.add({ title: 'Помилка видалення', color: 'error' })
  }
}

const getRowItems = (row) => [
  { type: 'label', label: row.title, class: 'truncate max-w-[200px]' },
  { type: 'separator' },
  { label: 'Переглянути', to: `/blog-posts/${row.id}` },
  { label: 'Редагувати', to: `/blog-posts/${row.id}/edit` },
  { type: 'separator' },
  { label: 'Видалити', icon: 'i-lucide-trash-2', color: 'error', onSelect: () => deletePost(row.id) }
]

const formatDate = (date) => {
  if (!date) return '—'
  return new Date(date).toLocaleDateString('uk-UA', {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  })
}

const declination = (n, forms) => {
  n = Math.abs(n) % 100
  const n1 = n % 10
  if (n > 10 && n < 20) return forms[2]
  if (n1 > 1 && n1 < 5) return forms[1]
  if (n1 === 1) return forms[0]
  return forms[2]
}

onMounted(() => {
  fetchPosts()
})

watch(() => useRoute().fullPath, () => {
  fetchPosts(paginationMeta.value.current_page)
})
</script>

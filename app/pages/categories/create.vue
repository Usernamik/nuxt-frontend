<template>
  <div class="p-6 max-w-xl mx-auto space-y-6">
    <!-- Header -->
    <div class="flex items-center gap-4">
      <UButton
        to="/categories"
        color="neutral"
        variant="ghost"
        size="sm"
        class="rounded-full"
      >
        <template #leading>
          <Icon name="i-lucide-arrow-left" class="w-4 h-4" />
        </template>
        Назад
      </UButton>
      <div>
        <h1 class="text-3xl font-bold tracking-tight">Створення категорії</h1>
        <p class="text-sm text-muted mt-1">Додайте нову категорію для постів блогу</p>
      </div>
    </div>

    <!-- Form Card -->
    <UCard variant="outline">
      <UForm :schema="schema" :state="state" class="space-y-5" @submit="onSubmit">
        <UFormField label="Назва" name="title">
          <UInput
            v-model="state.title"
            placeholder="Введіть назву категорії"
            size="lg"
            class="w-full"
          >
            <template #leading>
              <Icon name="i-lucide-folder" class="w-4 h-4 text-muted" />
            </template>
          </UInput>
        </UFormField>

        <UFormField label="Slug" name="slug" hint="залиште порожнім для автоматичної генерації">
          <UInput
            v-model="state.slug"
            placeholder="category-slug"
            class="w-full"
          >
            <template #leading>
              <Icon name="i-lucide-link" class="w-4 h-4 text-muted" />
            </template>
          </UInput>
        </UFormField>

        <UFormField label="Батьківська категорія" name="parent_id" hint="залиште порожнім для кореневої категорії">
          <USelect
            v-model="state.parent_id"
            :items="categoryOptions"
            placeholder="Немає батьківської"
            class="w-full"
          />
        </UFormField>

        <USeparator />

        <div class="flex items-center gap-3 justify-end">
          <UButton
            to="/categories"
            color="neutral"
            variant="outline"
          >
            Скасувати
          </UButton>
          <UButton
            type="submit"
            color="primary"
            size="lg"
            :loading="loading"
          >
            <template #leading>
              <Icon name="i-lucide-check" class="w-4 h-4" />
            </template>
            Створити категорію
          </UButton>
        </div>
      </UForm>
    </UCard>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import * as z from 'zod'

const router = useRouter()
const toast = useToast()
const loading = ref(false)
const allCategories = ref([])

const schema = z.object({
  title: z.string().min(2, 'Мінімум 2 символи'),
  slug: z.string().optional(),
  parent_id: z.number().nullable().optional()
})

const state = reactive({
  title: '',
  slug: '',
  parent_id: null
})

const categoryOptions = computed(() =>
  allCategories.value.map(c => ({ label: c.title, value: c.id }))
)

const fetchCategories = async () => {
  try {
    const res = await fetch('/api/admin/blog/categories')
    if (!res.ok) {
      console.error('[Category Create] Fetch categories failed:', res.status)
      return
    }
    const data = await res.json()
    allCategories.value = data.data || data
  } catch (err) {
    console.error('[Category Create] Error fetching categories:', err)
  }
}

const onSubmit = async (event) => {
  loading.value = true
  try {
    // Build payload — send null parent_id explicitly if not set
    const payload = { ...event.data, parent_id: event.data.parent_id ?? null }

    const res = await fetch('/api/admin/blog/categories', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    })
    if (!res.ok) {
      const errorText = await res.text().catch(() => '')
      console.error('[Category Create] Server responded:', res.status, errorText)
      throw new Error('Помилка сервера: ' + res.status)
    }
    toast.add({ title: 'Категорію створено', color: 'success' })
    router.push('/categories')
  } catch (err) {
    console.error('[Category Create] Error:', err)
    toast.add({ title: 'Помилка створення', description: err.message, color: 'error' })
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchCategories()
})
</script>

<template>
  <div class="p-6 max-w-3xl mx-auto space-y-6">
    <!-- Header -->
    <div class="flex items-center gap-4">
      <UButton
        to="/blog-posts"
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
        <h1 class="text-3xl font-bold tracking-tight">Створення поста</h1>
        <p class="text-sm text-muted mt-1">Заповніть форму щоб додати новий пост</p>
      </div>
    </div>

    <!-- Form Card -->
    <UCard variant="outline">
      <UForm :schema="schema" :state="state" class="space-y-6" @submit="onSubmit">
        <UFormField label="Заголовок" name="title">
          <UInput
            v-model="state.title"
            placeholder="Введіть заголовок поста"
            size="lg"
            class="w-full"
          >
            <template #leading>
              <Icon name="i-lucide-heading" class="w-4 h-4 text-muted" />
            </template>
          </UInput>
        </UFormField>

        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <UFormField label="Slug" name="slug" hint="залиште порожнім для автоматичної генерації">
            <UInput
              v-model="state.slug"
              placeholder="my-post-slug"
              class="w-full"
            >
              <template #leading>
                <Icon name="i-lucide-link" class="w-4 h-4 text-muted" />
              </template>
            </UInput>
          </UFormField>

          <UFormField label="Категорія" name="category_id">
            <USelect
              v-model="state.category_id"
              :items="categoryOptions"
              placeholder="Оберіть категорію"
              class="w-full"
            />
          </UFormField>
        </div>

        <UFormField label="Контент" name="content_raw">
          <UTextarea
            v-model="state.content_raw"
            placeholder="Напишіть вміст поста..."
            :rows="12"
            class="w-full"
          />
        </UFormField>

        <USeparator />

        <div class="flex items-center justify-between">
          <UCheckbox v-model="state.is_published" label="Опублікувати відразу" size="lg" />
        </div>

        <USeparator />

        <div class="flex items-center gap-3 justify-end">
          <UButton
            to="/blog-posts"
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
            Створити пост
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
const categories = ref([])
const categoryOptions = computed(() =>
  categories.value.map(c => ({ label: c.title, value: c.id }))
)

const schema = z.object({
  title: z.string().min(5, 'Мінімум 5 символів'),
  slug: z.string().optional(),
  category_id: z.number({ invalid_type_error: 'Оберіть категорію' }).nullable().optional(),
  content_raw: z.string().min(5, 'Мінімум 5 символів'),
  is_published: z.boolean().default(false)
})

const state = reactive({
  title: '',
  slug: '',
  category_id: null,
  content_raw: '',
  is_published: false
})

const fetchCategories = async () => {
  try {
    const res = await fetch('/api/admin/blog/categories')
    if (!res.ok) {
      console.error('[Post Create] Categories fetch failed:', res.status)
      throw new Error('Помилка сервера: ' + res.status)
    }
    const data = await res.json()
    categories.value = data.data || data
    console.log('[Post Create] Categories loaded:', categories.value.length)
  } catch (err) {
    console.error('[Post Create] Error loading categories:', err)
    toast.add({ title: 'Помилка завантаження категорій', description: err.message, color: 'error' })
  }
}

const onSubmit = async (event) => {
  loading.value = true
  try {
    const res = await fetch('/api/admin/blog/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(event.data)
    })
    if (!res.ok) {
      const errorBody = await res.text().catch(() => '')
      console.error('[Post Create] Server responded:', res.status, errorBody)
      // Try to extract message from Laravel JSON response
      let msg = 'Помилка сервера: ' + res.status
      try {
        const json = JSON.parse(errorBody)
        if (json.message) msg = json.message
      } catch {}
      throw new Error(msg)
    }
    toast.add({ title: 'Пост створено', color: 'success' })
    router.push('/blog-posts')
  } catch (err) {
    toast.add({ title: 'Помилка створення', description: err.message, color: 'error' })
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchCategories()
})
</script>

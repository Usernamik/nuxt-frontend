<template>
  <div class="p-6 max-w-4xl mx-auto space-y-6">
    <!-- Back Button -->
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
      До списку постів
    </UButton>

    <!-- Loading State -->
    <div v-if="pending" class="space-y-6 animate-pulse">
      <div class="h-10 w-3/4 rounded-lg bg-muted" />
      <div class="flex gap-3">
        <div class="h-6 w-24 rounded-full bg-muted" />
        <div class="h-6 w-32 rounded-full bg-muted" />
      </div>
      <div class="space-y-3">
        <div v-for="n in 6" :key="n" class="h-4 rounded bg-muted" :class="n % 2 ? 'w-full' : 'w-5/6'" />
      </div>
    </div>

    <!-- Error State -->
    <div v-else-if="!post" class="flex flex-col items-center justify-center py-20 text-center">
      <div class="w-16 h-16 rounded-2xl bg-muted flex items-center justify-center mb-4">
        <Icon name="i-lucide-file-x" class="w-8 h-8 text-muted" />
      </div>
      <h3 class="text-lg font-semibold">Пост не знайдено</h3>
      <p class="text-sm text-muted mt-1">Можливо, його було видалено або він не існує</p>
    </div>

    <!-- Post Content -->
    <article v-else class="space-y-6">
      <!-- Meta Card -->
      <UCard variant="outline" class="overflow-hidden">
        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-4">
          <h1 class="text-3xl font-bold tracking-tight">{{ post.title }}</h1>
          <div class="flex items-center gap-2 shrink-0">
            <UTooltip text="Редагувати">
              <UButton
                :to="`/blog-posts/${post.id}/edit`"
                color="primary"
                variant="outline"
                size="sm"
              >
                <template #leading>
                  <Icon name="i-lucide-pencil" class="w-4 h-4" />
                </template>
                Редагувати
              </UButton>
            </UTooltip>
          </div>
        </div>

        <div class="flex flex-wrap items-center gap-3 text-sm text-muted">
          <UBadge
            :color="post.is_published ? 'success' : 'neutral'"
            :variant="post.is_published ? 'subtle' : 'outline'"
            size="sm"
          >
            {{ post.is_published ? 'Опубліковано' : 'Чернетка' }}
          </UBadge>

          <span class="flex items-center gap-1">
            <Icon name="i-lucide-calendar" class="w-3.5 h-3.5" />
            {{ formatDate(post.published_at || post.created_at) }}
          </span>

          <span v-if="post.category" class="flex items-center gap-1">
            <Icon name="i-lucide-folder" class="w-3.5 h-3.5" />
            {{ post.category.title }}
          </span>

          <span v-if="post.user" class="flex items-center gap-1">
            <Icon name="i-lucide-user" class="w-3.5 h-3.5" />
            {{ post.user.name }}
          </span>

          <span class="flex items-center gap-1">
            <Icon name="i-lucide-hash" class="w-3.5 h-3.5" />
            ID: {{ post.id }}
          </span>
        </div>
      </UCard>

      <!-- Content Card -->
      <UCard variant="outline">
        <div class="prose prose-neutral dark:prose-invert max-w-none">
          <div class="whitespace-pre-wrap text-base leading-relaxed">
            {{ post.content_raw }}
          </div>
        </div>
      </UCard>

      <!-- Actions -->
      <div class="flex items-center gap-3">
        <UButton
          to="/blog-posts"
          color="neutral"
          variant="outline"
        >
          <template #leading>
            <Icon name="i-lucide-arrow-left" class="w-4 h-4" />
          </template>
          До списку
        </UButton>
        <UButton
          :to="`/blog-posts/${post.id}/edit`"
          color="primary"
        >
          <template #leading>
            <Icon name="i-lucide-pencil" class="w-4 h-4" />
          </template>
          Редагувати
        </UButton>
      </div>
    </article>
  </div>
</template>

<script setup>
const route = useRoute()
const { data: response, pending } = await useFetch(`/api/admin/blog/posts/${route.params.id}`)
const post = computed(() => response.value?.data || response.value)

const formatDate = (date) => {
  if (!date) return '—'
  return new Date(date).toLocaleDateString('uk-UA', {
    day: 'numeric',
    month: 'long',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}
</script>
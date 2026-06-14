<template>
  <div class="p-6 max-w-4xl mx-auto space-y-8">
    <!-- Hero Section -->
    <div class="text-center pt-12 pb-8">
      <div class="w-16 h-16 rounded-2xl bg-primary/10 flex items-center justify-center mx-auto mb-6">
        <Icon name="i-lucide-newspaper" class="w-8 h-8 text-primary" />
      </div>
      <h1 class="text-3xl font-bold tracking-tight">Панель керування блогом</h1>
      <p class="text-muted mt-2 max-w-md mx-auto">
        Керуйте постами та категоріями вашого блогу в зручному інтерфейсі
      </p>
    </div>

    <!-- Navigation Cards -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <NuxtLink to="/blog-posts" class="group block h-full">
        <UCard
          variant="outline"
          class="h-full transition-all duration-200 hover:border-primary hover:shadow-md hover:shadow-primary/5"
        >
          <div class="flex items-start gap-4">
            <div class="w-12 h-12 rounded-xl bg-primary/10 flex items-center justify-center shrink-0 group-hover:scale-110 transition-transform duration-200">
              <Icon name="i-lucide-file-text" class="w-6 h-6 text-primary" />
            </div>
            <div class="flex-1 min-w-0">
              <div class="flex items-center gap-2">
                <h2 class="text-lg font-semibold group-hover:text-primary transition-colors">
                  Пости блогу
                </h2>
                <Icon name="i-lucide-arrow-right" class="w-4 h-4 text-muted group-hover:text-primary group-hover:translate-x-1 transition-all" />
              </div>
              <p class="text-sm text-muted mt-1">
                Створюйте, редагуйте та керуйте публікаціями. Додавайте контент, категорії та відстежуйте статус.
              </p>
            </div>
          </div>
        </UCard>
      </NuxtLink>

      <NuxtLink to="/categories" class="group block h-full">
        <UCard
          variant="outline"
          class="h-full transition-all duration-200 hover:border-primary hover:shadow-md hover:shadow-primary/5"
        >
          <div class="flex items-start gap-4">
            <div class="w-12 h-12 rounded-xl bg-primary/10 flex items-center justify-center shrink-0 group-hover:scale-110 transition-transform duration-200">
              <Icon name="i-lucide-folder-tree" class="w-6 h-6 text-primary" />
            </div>
            <div class="flex-1 min-w-0">
              <div class="flex items-center gap-2">
                <h2 class="text-lg font-semibold group-hover:text-primary transition-colors">
                  Категорії
                </h2>
                <Icon name="i-lucide-arrow-right" class="w-4 h-4 text-muted group-hover:text-primary group-hover:translate-x-1 transition-all" />
              </div>
              <p class="text-sm text-muted mt-1">
                Організовуйте пости за темами. Додавайте, редагуйте та впорядковуйте категорії блогу.
              </p>
            </div>
          </div>
        </UCard>
      </NuxtLink>
    </div>

    <!-- Stats / Info Section -->
    <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 pt-4">
      <div class="flex items-center gap-3 p-4 rounded-xl bg-muted/50">
        <div class="w-10 h-10 rounded-lg bg-primary/10 flex items-center justify-center">
          <Icon name="i-lucide-file-text" class="w-5 h-5 text-primary" />
        </div>
        <div>
          <p class="text-xs text-muted uppercase tracking-wide">Пости</p>
          <p class="text-lg font-semibold">CRUD</p>
        </div>
      </div>
      <div class="flex items-center gap-3 p-4 rounded-xl bg-muted/50">
        <div class="w-10 h-10 rounded-lg bg-primary/10 flex items-center justify-center">
          <Icon name="i-lucide-folder-tree" class="w-5 h-5 text-primary" />
        </div>
        <div>
          <p class="text-xs text-muted uppercase tracking-wide">Категорії</p>
          <p class="text-lg font-semibold">CRUD</p>
        </div>
      </div>
      <div class="flex items-center gap-3 p-4 rounded-xl bg-muted/50">
        <div class="w-10 h-10 rounded-lg bg-primary/10 flex items-center justify-center">
          <Icon name="i-lucide-layout-dashboard" class="w-5 h-5 text-primary" />
        </div>
        <div>
          <p class="text-xs text-muted uppercase tracking-wide">Стек</p>
          <p class="text-lg font-semibold">Nuxt UI</p>
        </div>
      </div>
    </div>
  </div>
</template>

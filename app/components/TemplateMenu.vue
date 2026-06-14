<template>
  <UDropdownMenu
    v-slot="{ open }"
    :modal="false"
    :items="menuItems"
    :content="{ align: 'start' }"
    :ui="{ content: 'min-w-fit' }"
    size="xs"
  >
    <UButton
      label="Меню"
      variant="subtle"
      trailing-icon="i-lucide-chevron-down"
      size="xs"
      class="-mb-[6px] font-semibold rounded-full truncate"
      :class="[open && 'bg-primary/15']"
      :ui="{
        trailingIcon: ['transition-transform duration-200', open ? 'rotate-180' : undefined].filter(Boolean).join(' ')
      }"
    />
  </UDropdownMenu>
</template>

<script setup>
const menuItems = [
  {
    label: 'Блог',
    icon: 'i-lucide-file-text',
    children: [
      { label: 'Пости', to: '/blog-posts', icon: 'i-lucide-file-text' },
      { label: 'Категорії', to: '/categories', icon: 'i-lucide-folder' }
    ]
  },
  { type: 'separator' },
  {
    label: 'Шаблони',
    icon: 'i-lucide-template',
    children: [
      {
        label: 'Starter',
        to: 'https://starter-template.nuxt.dev/',
        color: 'primary',
        checked: true,
        type: 'checkbox'
      },
      { label: 'Landing', to: 'https://landing-template.nuxt.dev/' },
      { label: 'Docs', to: 'https://docs-template.nuxt.dev/' },
      { label: 'SaaS', to: 'https://saas-template.nuxt.dev/' },
      { label: 'Dashboard', to: 'https://dashboard-template.nuxt.dev/' },
      { label: 'Chat', to: 'https://chat-template.nuxt.dev/' },
      { label: 'Portfolio', to: 'https://portfolio-template.nuxt.dev/' },
      { label: 'Changelog', to: 'https://changelog-template.nuxt.dev/' }
    ]
  }
]
</script>

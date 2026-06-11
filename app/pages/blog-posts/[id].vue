<template>
  <div>
    <div v-if="pending">Завантаження...</div>
    <div v-else-if="post">
      <h1>Пост ID: {{ post.id }}</h1>
      <h2>{{ post.title }}</h2>
      <p>{{ post.content_raw }}</p>
      <p>Дата: {{ post.date_published || post.published_at }}</p>
      <NuxtLink to="/blog-posts">Назад до списку</NuxtLink>
    </div>
    <div v-else>Помилка завантаження</div>
  </div>
</template>

<script setup>
const route = useRoute()
const { data: response, pending } = await useFetch(`http://localhost/api/admin/blog/posts/${route.params.id}`)
const post = response.value?.data || response.value
</script>
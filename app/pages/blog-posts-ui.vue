<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">Пости блогу</h1>
    
    <table border="1" cellpadding="8" cellspacing="0" style="width: 100%; border-collapse: collapse;">
      <thead>
        <tr style="background: #f0f0f0;">
          <th>#</th>
          <th>Заголовок</th>
          <th>Дата публікації</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="post in paginatedPosts" :key="post.id">
          <td>{{ post.id }}</td>
          <td>{{ post.title }}</td>
          <td>{{ post.published_at }}</td>
        </tr>
      </tbody>
    </table>
    
    <div class="flex justify-center mt-4" style="display: flex; gap: 10px; margin-top: 20px;">
      <button 
        @click="currentPage--" 
        :disabled="currentPage === 1"
        style="padding: 5px 10px; cursor: pointer;"
      >
        Попередня
      </button>
      <span>Сторінка {{ currentPage }} з {{ totalPages }}</span>
      <button 
        @click="currentPage++" 
        :disabled="currentPage === totalPages"
        style="padding: 5px 10px; cursor: pointer;"
      >
        Наступна
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const posts = ref([])
const currentPage = ref(1)
const perPage = ref(10)

const totalPages = computed(() => Math.ceil(posts.value.length / perPage.value))

const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * perPage.value
  const end = start + perPage.value
  return posts.value.slice(start, end)
})

const fetchPosts = async () => {
  try {
    const response = await fetch('http://localhost/api/blog/posts')
    const data = await response.json()
    posts.value = Array.isArray(data) ? data : (data.data || [])
  } catch (error) {
    console.error('Помилка:', error)
  }
}

onMounted(() => {
  fetchPosts()
})
</script>
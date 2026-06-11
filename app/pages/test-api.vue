<template>
  <div>
    <h1>Тест API</h1>
    <pre>{{ JSON.stringify(posts, null, 2) }}</pre>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const posts = ref([])

onMounted(async () => {
  try {
    const res = await fetch('http://localhost/api/blog/posts')
    const data = await res.json()
    console.log('API Response:', data)
    posts.value = data
  } catch (err) {
    console.error('Error:', err)
  }
})
</script>
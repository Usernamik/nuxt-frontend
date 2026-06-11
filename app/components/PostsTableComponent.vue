<template>
    <div>
        <table border="1">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Автор</th>
                    <th>Категорія</th>
                    <th>Заголовок</th>
                    <th>Дата публікації</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="post in posts" :key="post.id">
                    <td>{{ post.id }}</td>
                    <td>{{ post.user?.name || 'Невідомо' }}</td>
                    <td>{{ post.category?.title || 'Без категорії' }}</td>
                    <td>{{ post.title }}</td>
                    <td>{{ post.published_at }}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const posts = ref([])

onMounted(() => {
    fetch('http://localhost/api/blog/posts')
        .then(res => res.json())
        .then(data => {
            posts.value = data.data || data
        })
        .catch(err => console.error(err))
})
</script>
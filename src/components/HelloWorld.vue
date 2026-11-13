<template>
  <div>
    <h1>Posts</h1>
    <div v-if="loading">Loading posts...</div>
    <ul v-else>
      <li v-for="post in posts" :key="post.id">
        <h3>{{ post.title }}</h3>
        <p>{{ post.body }}</p>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import api from '../services/api'; // Import the global Axios instance

const posts = ref([]);
const loading = ref(true);

const fetchPosts = async () => {
  try {
    const response = await api.get('/posts');
    posts.value = response.data; // Assign JSON payload to the reactive variable
  } catch (error) {
    console.error('Error fetching data from Laravel:', error);
  } finally {
    loading = ref(false);
  }
};

// Fetch data automatically when the component mounts
onMounted(() => {
  fetchPosts();
});
</script>

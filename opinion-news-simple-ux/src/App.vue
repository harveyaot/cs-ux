<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'

// define const posts is list and error
const posts = ref([])
const error = ref(null)
// a function to parse date string to better format
const date = (dateString) => {
  const date = new Date(dateString)
  const options = {
    hour: 'numeric',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }
  return new Intl.DateTimeFormat('en-US', options).format(date)
}
// write mounted method calling axios get method, and assign response to posts
onMounted(async () => {
  try {
    // calling strapi api sort by updateDate in descending order
    const response = await axios.get('https://cms-strapi-longriver.azurewebsites.net/api/opinoin-news-many?sort[0]=updateDate:desc')
    posts.value = response.data.data

  } catch (error) {
    error = error;
  }
})
</script>

<template>
  <div id="app">
    <div v-if="error">
      {{ error }}
    </div>
    <ul v-else>
      <li v-for="post in posts" :key="post.id">
        <a :href="post.attributes.url" target="_blank">
        {{ post.attributes.title }}</a>

        <!--ouput post summay-->
        <p>{{ post.attributes.summary }}</p>
        
        <!--output author with bold font-->
        <p><strong>Author: {{ post.attributes.author }}</strong></p>

        <p><strong>Source: {{ post.attributes.source }}</strong></p>

        <!--parse post updateDate and output in good style-->
        <p><strong>{{ date(post.attributes.updateDate) }} </strong></p>

        <!--render post thumbnail-->
        <img :src="post.attributes.thumbnail" alt="thumbnail" />


      </li>
    </ul>
  </div>
</template>
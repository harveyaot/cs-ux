<script setup>
import axios from 'axios'
import { onMounted, ref, watch} from 'vue'

// define const posts is list and error
const posts = ref([])
const error = ref(null)
const PageLimit = 20;
const publishers = ref(['all', 'newsweek', 'bloomberg', 'scmp', 'foreignpolicy'])
const selectedPublisher = ref('all')
const CMSBaseUrl = 'https://cms-strapi-longriver.azurewebsites.net/api/opinoin-news-many'
// add pagination infomation
const pagination = ref({
  page: 1,
  pageSize: 10,
  pageCount: 0,                       
  total: 0,
})


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
    // calling strapi api sort by updateDate in descending order with page parameter
    const response = await axios.get(CMSBaseUrl + '?sort[0]=updateDate:desc&pagination[pageSize]=' + pagination.value.pageSize)
    posts.value = response.data.data
    // load pagination information from response.data.meta
    pagination.value = response.data.meta.pagination

  } catch (error) {
    error = error;
  }
})

// add watch function on pagenation.page to call axios get method when page change
watch(() => pagination.value.page, async () => {
  try {
    var url = CMSBaseUrl + '?sort[0]=updateDate:desc&pagination[page]=' + pagination.value.page + '&pagination[pageSize]=' + pagination.value.pageSize
    if (selectedPublisher.value != 'all'){
      url += `&filters[source][$eq]=${selectedPublisher.value}`
    }
    // calling strapi api sort by updateDate in descending order with page parameter
    const response = await axios.get(url)
    //const response = await axios.get('https://cms-strapi-longriver.azurewebsites.net/api/opinoin-news-many?sort[-1]=updateDate:desc')
    posts.value = response.data.data
    // load pagination information from response.data.meta
    pagination.value = response.data.meta.pagination

  } catch (error) {
    error = error;
  }
})

watch(selectedPublisher, async (newPublisher) => {
  try {
    pagination.value.page = 1
    var url = CMSBaseUrl + '?sort[0]=updateDate:desc&pagination[page]=' + pagination.value.page + '&pagination[pageSize]=' + pagination.value.pageSize
    if (newPublisher != 'all'){
      url += `&filters[source][$eq]=${newPublisher}`
    }
    // calling strapi api sort by updateDate in descending order with page parameter
    const response = await axios.get(url)
    //const response = await axios.get('https://cms-strapi-longriver.azurewebsites.net/api/opinoin-news-many?sort[-1]=updateDate:desc')
    posts.value = response.data.data
    // load pagination information from response.data.meta
    pagination.value = response.data.meta.pagination

  } catch (error) {
    error = error;
  }
})
</script>

<template>
  <div v-if="posts && posts.length > 0" id="app">
    <h1 posts> Opinions Collection </h1>
    <hr>
    <br>
    <div>
      <b>Filter Publisher: </b>
      <select v-model="selectedPublisher">
        <option v-for="(choice, index) in publishers" :key="index" :value="choice">
          {{ choice }}
        </option>
      </select>
    </div>
    <br>
    <div v-if="error">
      {{ error }}
    </div>
    <ul v-else>
      <li v-for="post in posts" :key="post.id">
        <div class="article">
          <a :href="post.attributes.url" target="_blank" class=title>
          {{ post.attributes.title }}</a>

          
          <!--output author with bold font-->
          <p class="side-info"> 
              <span class="key">By: </span> 
              <span>{{ post.attributes.author }} </span>
              &nbsp;
              <span class="key">From: </span>
              <span>{{ post.attributes.source }}</span>
              &nbsp;
              <span class="key">Date: </span>
              <span>{{ date(post.attributes.updateDate) }}</span>
          </p>

          <!--ouput post summay-->
          <p class="summary">{{ post.attributes.summary }}</p>

          <!--render post thumbnail with fixed width and height, center-->
          <img class=th :src="post.attributes.thumbnail" alt="thumbnail" width="200" height="150" />
          <!-- <img :src="post.attributes.thumbnail" alt="thumbnail" />-->
        </div>
        <br>
      </li>
      
    </ul>

    <!--insert a seperate line, with some spaces from above-->
    <hr>

    <!--add pagination ux from pagination info-->
    <div  v-if="pagination.pageCount > 1">
      <button class="pagenav prev" v-if="pagination.page > 1" @click="pagination.page--">Prev</button>
      <!--add a place show the current page-->
      <span>Page {{ pagination.page }} of {{ Math.min(pagination.pageCount, PageLimit) }}</span>
      <!--show next page button when page less than pageCount and less than pageLimit-->
      <button class="pagenav next" v-if="pagination.page < pagination.pageCount && pagination.page < Math.min(pagination.pageCount, PageLimit)" @click="pagination.page++">Next</button>
    </div>
  </div>
</template>
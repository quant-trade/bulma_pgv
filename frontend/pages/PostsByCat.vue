<template>
<div class="container">
  <div class="columns">
    <div class="column is-two-third">
      <ad-component></ad-component>
      <h1 class="title is-1">{{ catTitle }}<span v-if="page > 0">, page {{ page }}</span></h1>
      <div class="columns" v-for="(chunk, index) in chunkPosts" :key="'p-' + index">
        <div class="column is-half" v-for="(post, i) in chunk" :key="index + i">
          <div class="card">
            <div class="card-image" v-if="post.image">
              <figure class="image is-4by3">
                <a :href="baseUrl + post.slug + '/'"><img :src="imgBaseUrl + post.image" :alt="post.title"></a>
              </figure>
            </div>
            <div class="card-content">
              <p class="title is-5">
                <a :href="baseUrl + keyword + '/' + post.category_id.Slug + '/'">{{ post.category_id.Title }}</a>
                  | {{ post.date | formatDate }}
              </p>
              <h1 class="title is-3"><a :href="baseUrl + post.slug + '/'">{{ post.title }}</a></h1>
              <div class="content">
                <p v-if="post.content">{{ post.content | truncate }}</p>
              </div>
            </div>
          </div>
        </div>
        <div class="column is-half" v-if="index === (3 || 7)">
          <ad-component></ad-component>
        </div>
      </div>
      <paginator-component v-once :totalPages="calcPages" :paginatorType="paginatorType" :value="catSlug" :currentPage="page" :itemsPerPage="itemsPerPage" :totalItems="posts[0].total_posts">
      </paginator-component>
    </div>
  </div>
</div>
</template>

<script>
import chunk from '../plugins/chunk'
import Paginator from '../components/Paginator.vue'
import Ads from '../components/Ads.vue'
import axios from 'axios'

export default {
  data () {
    return {
      posts: [],
      baseUrl: process.env.BASE_URL,
      imgBaseUrl: process.env.IMG_URL,
      title: process.env.SITE_NAME,
      keyword: process.env.KEYWORD,
      page: null,
      paginatorType: 1,
      itemsPerPage: 20,
      catSlug: null,
      catTitle: null
    }
  },
  asyncData ({ req, params, error }) {
    return axios.get('/cat/' + params.catSlug + '/' + (Number(params.page) || '0') + '/')
      .then((response) => {
        return { posts: response.data, catSlug: params.catSlug, catTitle: response.data[0].category_id.Title, page: Number(params.page) || 0 }
      })
      .catch((e) => {
        error({ statusCode: 500, message: e })
      })
  },
  components: {
    'paginator-component': Paginator,
    'ad-component': Ads
  },
  computed: {
    chunkPosts () {
      return chunk(this.posts, 2)
    },
    calcPages () {
      const pages = Math.floor(this.posts[0].total_posts / 20)
      return pages <= 250 ? pages : 250
    }
  },
  head () {
    return {
      title: Number(this.$route.params.page) ? 'Page ' + Number(this.$route.params.page) + ' ' + this.catTitle + ' | ' + this.title : this.title
    }
  }
}
</script>

<style>
</style>
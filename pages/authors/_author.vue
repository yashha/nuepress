<template>
  <div>
    <h1 v-html="author.name"></h1>
    <p v-html="author.description"></p>
    <ArticleList :articles="authorArticles.articles"></ArticleList>
  </div>
</template>

<script>
import find from 'lodash/find'
import axios from 'axios'
import ArticleList from '~/components/ArticleList'

export default {
  async asyncData ({ store, params }) {
    if (!store.state.featuredArticles.length) {
      let articles = await axios.get(`${store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&categories=${store.state.featuredID}&_embed`)
      store.commit('setFeaturedArticles', articles.data)
    }

    if (!store.state.authors) {
      let authors = await axios.get(`${store.state.wordpressAPI}/wp/v2/users?per_page=100`)
      store.commit('setAuthors', authors.data)
    }

    if (!find(store.state.authorArticles, {'slug': params.author})) {
      let author = find(store.state.authors, {'slug': params.author})
      let authorArticles = await axios.get(`${store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&author=${author.id}&_embed`)
      store.commit('setAuthorArticles', {slug: params.author, articles: authorArticles.data, infiniteLoading: true, page: 1})
    }

    if (!store.state.meta) {
      let meta = await axios.get(store.state.wordpressAPI)
      store.commit('setMeta', meta.data)
    }
  },

  components: {
    ArticleList
  },

  computed: {
    author () {
      return find(this.$store.state.authors, {
        'slug': this.$route.params.author
      })
    },
    authorArticles () {
      return find(this.$store.state.authorArticles, {
        'slug': this.$route.params.author
      })
    },
    authors () {
      return this.$store.state.authors
    },
    meta () {
      return this.$store.state.meta
    }
  },

  head () {
    return {
      title: `${this.author.name} | ${this.meta.name}`,
      meta: [
        { description: this.meta.description }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

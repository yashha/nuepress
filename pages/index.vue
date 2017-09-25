<template>
  <div>
    <ArticleList :articles="articles"/>
  </div>
</template>

<script>
import axios from 'axios'
import ArticleList from '~/components/ArticleList'

export default {
  async asyncData ({ store, params }) {
    if (!store.state.articles.length) {
      let articles = await axios.get(`${store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&categories_exclude=${store.state.featuredID}&_embed`)
      store.commit('setArticles', articles.data)
    }

    if (!store.state.featuredArticles.length) {
      let articles = await axios.get(`${store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&categories=${store.state.featuredID}&_embed`)
      store.commit('setFeaturedArticles', articles.data)
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
    articles () {
      return this.$store.state.articles
    },
    meta () {
      return this.$store.state.meta
    }
  },

  head () {
    return {
      title: `Home | ${this.meta.name}`,
      meta: [
        { description: this.meta.description }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

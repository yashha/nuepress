<template>
  <div>
    <h1 v-html="topic.name"></h1>
    <p v-html="topic.description"></p>
    <ArticleList :articles="topicArticles.articles"></ArticleList>
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

    if (!store.state.topics) {
      let topics = await axios.get(`${store.state.wordpressAPI}/wp/v2/categories?per_page=100`)
      store.commit('setTopics', topics.data)
    }

    if (!find(store.state.topicArticles, {'slug': params.topic})) {
      let topic = find(store.state.topics, {'slug': params.topic})
      let topicArticles = await axios.get(`${store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&categories=${topic.id}&_embed`)
      store.commit('setTopicArticles', {slug: params.topic, articles: topicArticles.data, infiniteLoading: true, page: 1})
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
    meta () {
      return this.$store.state.meta
    },
    topic () {
      return find(this.$store.state.topics, {
        'slug': this.$route.params.topic
      })
    },
    topicArticles () {
      return find(this.$store.state.topicArticles, {
        'slug': this.$route.params.topic
      })
    },
    topics () {
      return this.$store.state.topic
    }
  },

  head () {
    return {
      title: `${this.topic.name} | ${this.meta.name}`,
      meta: [
        { description: this.meta.description }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

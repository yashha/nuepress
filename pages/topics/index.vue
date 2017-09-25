<template>
  <div class="page">
    <div class="page-title">
      <h1>Topics</h1>
    </div>
    <ul>
      <li v-for="topic in topics" v-if="topic.slug !== 'featured'">
        <nuxt-link :to="`/topics/${topic.slug}`">
          <h2 v-html="topic.name"></h2>
          <div v-html="topic.description"></div>
        </nuxt-link>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  async asyncData ({ store, params }) {
    if (!store.state.topics) {
      let topics = await axios.get(`${store.state.wordpressAPI}/wp/v2/categories?per_page=100`)
      store.commit('setTopics', topics.data)
    }

    if (!store.state.meta) {
      let meta = await axios.get(store.state.wordpressAPI)
      store.commit('setMeta', meta.data)
    }
  },

  computed: {
    meta () {
      return this.$store.state.meta
    },
    topics () {
      return this.$store.state.topics
    }
  },

  head () {
    return {
      title: `Topics | ${this.meta.name}`,
      meta: [
        { description: this.meta.description }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

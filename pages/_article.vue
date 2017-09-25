<template>
  <article>
    <img :src="featuredImage.source_url">
    <h1>{{ article.title.rendered }}</h1>
    <div>{{ article.date }}</div>
    <nuxt-link :to="`/authors/${author.slug}`">{{ author.name }}</nuxt-link>
    <div v-html="article.content.rendered"></div>
  </article>
</template>

<script>
import axios from 'axios'

export default {
  async asyncData ({ store, params }) {
    let article = await axios.get(`${store.state.wordpressAPI}/wp/v2/posts?slug=${params.article}&_embed`)
    store.commit('setArticle', article.data[0])

    if (!store.state.meta) {
      let meta = await axios.get(store.state.wordpressAPI)
      store.commit('setMeta', meta.data)
    }
  },

  computed: {
    article () {
      return this.$store.state.article
    },
    author () {
      return this.$store.state.article._embedded.author[0]
    },
    featuredImage () {
      let featuredImage = this.$store.state.article._embedded['wp:featuredmedia']

      if (featuredImage) {
        return featuredImage[0].media_details.sizes.large || featuredImage[0].media_details.sizes.full || false
      } else {
        return { height: 0, width: 0 }
      }
    },
    meta () {
      return this.$store.state.meta
    }
  },

  head () {
    return {
      title: `${this.article.title.rendered} | ${this.meta.name}`,
      meta: [
        { description: this.article.excerpt.rendered }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

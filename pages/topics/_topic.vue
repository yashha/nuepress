<template>
  <div class="topic">
    <div class="articles">
      <div class="page-title">
        <h1>{{ topic.name }}</h1>
        <p v-if="topic.description">
          {{ topic.description }}
        </p>
      </div>
      <ArticleList :articles="topicArticles.articles" />
      <InfiniteLoading
        v-if="isLoadingMore"
        ref="infiniteLoading"
        :on-infinite="moreArticles"
      >
        <span slot="spinner">
          <Spinner1 />
        </span>
        <span slot="no-results">
          <Smile />
          <div>No more articles!</div>
        </span>
        <span slot="no-more">
          <Smile />
          <div>No more articles!</div>
        </span>
      </InfiniteLoading>
    </div>
    <TheSidebar :featured-articles="$store.state.featuredArticles" />
  </div>
</template>

<script>
import find from 'lodash/find'
import InfiniteLoading from 'vue-infinite-loading/src/components/InfiniteLoading.vue'
import ArticleList from '~/components/ArticleList'
import TheSidebar from '~/components/TheSidebar'
import Smile from '~/assets/svg/Smile.vue'
import Spinner1 from '~/components/Spinner1.vue'

export default {

  components: {
    ArticleList,
    TheSidebar,
    InfiniteLoading,
    Smile,
    Spinner1
  },

  computed: {
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
    isLoadingMore () {
      return this.topicArticles.infiniteLoading && this.topicArticles.articles.length >= 10
    }
  },
  async asyncData ({ app, store, params }) {
    if (!store.state.featuredArticles.length) {
      const articles = await app.$wp.posts(10).embed().categories(store.state.featuredID).embed()
      store.commit('setFeaturedArticles', articles)
    }

    if (!store.state.topics) {
      const topics = await app.$wp.categories().perPage(100)
      store.commit('setTopics', topics)
    }

    if (!find(store.state.topicArticles, { 'slug': params.topic })) {
      const topic = find(store.state.topics, { 'slug': params.topic })
      const topicArticles = await app.$wp.posts(10).embed().categories(topic.id).embed()
      store.commit('setTopicArticles', { slug: params.topic, articles: topicArticles, infiniteLoading: true, page: 1 })
    }
  },

  head () {
    return {
      title: `${this.topic.name} | ${this.$store.state.meta.name}`,
      meta: [
        { description: this.$store.state.meta.description }
      ]
    }
  },

  methods: {
    moreArticles () {
      this.topicArticles.page++

      this.$axios.get(`${this.$store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&categories=${this.topic.id}&_embed&page=${this.topicArticles.page}`)
        .then((response) => {
          this.topicArticles.articles = this.topicArticles.articles.concat(response.data)
          this.$refs.infiniteLoading.$emit('$InfiniteLoading:loaded')
        })
        .catch(() => {
          this.$refs.infiniteLoading.$emit('$InfiniteLoading:complete')
        })
    }
  }
}
</script>

<style lang="scss" scoped>
@import '~assets/css/vars.scss';

.topic {
  display: flex;

  .articles {
    background-color: #efefef;
    padding: 0 32px;
    max-width: 900px;
    width: 100%;

    @media (max-width: 1000px) {
      max-width: none;
    }

    @media (max-width: 700px) {
      padding: 0 16px;
    }

    .article-list {
      margin: 32px 0;
    }
  }
}
</style>

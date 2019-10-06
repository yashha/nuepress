<template>
  <div class="author">
    <div class="articles">
      <div class="page-title">
        <h1>{{ author.name }}</h1>
        <p v-if="author.description">
          {{ author.description }}
        </p>
      </div>
      <ArticleList :articles="authorArticles.articles" />
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
    isLoadingMore () {
      return this.authorArticles.infiniteLoading && this.authorArticles.articles.length >= 10
    }
  },
  async asyncData ({ app, store, params }) {
    if (!store.state.featuredArticles.length) {
      const articles = await app.$wp.posts(10).categories(store.state.featuredID).embed()
      store.commit('setFeaturedArticles', articles)
    }

    if (!store.state.authors) {
      const authors = await app.$wp.users(100).embed()
      store.commit('setAuthors', authors)
    }

    if (!find(store.state.authorArticles, { 'slug': params.author })) {
      const author = find(store.state.authors, { 'slug': params.author })
      const authorArticles = await app.$axios.get(`${store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&author=${author.id}&_embed`)
      store.commit('setAuthorArticles', { slug: params.author, articles: authorArticles.data, infiniteLoading: true, page: 1 })
    }
  },

  head () {
    return {
      title: `${this.author.name} | ${this.$store.state.meta.name}`,
      meta: [
        { description: this.$store.state.meta.description }
      ]
    }
  },

  methods: {
    moreArticles () {
      this.authorArticles.page++

      this.$axios.get(`${this.$store.state.wordpressAPI}/wp/v2/posts?orderby=date&per_page=10&author=${this.author.id}&_embed&page=${this.authorArticles.page}`)
        .then((response) => {
          this.authorArticles.articles = this.authorArticles.articles.concat(response.data)
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

.author {
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

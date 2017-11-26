<template>
  <div class="home">
    <div class="articles">
      <Hero :heroArticle="heroArticle" v-if="heroArticle"/>
      <ArticleList :articles="$store.state.articles"/>
      <InfiniteLoading v-if="indexInfiniteLoading.enabled" :on-infinite="moreArticles" ref="infiniteLoading"/>
    </div>
    <Sidebar :featuredArticles="$store.state.featuredArticles"/>
  </div>
</template>

<script>
import ArticleList from '~/components/ArticleList'
import Hero from '~/components/Hero'
import InfiniteLoading from '~/components/InfiniteLoading'
import Sidebar from '~/components/Sidebar'

export default {
  async asyncData ({ app, store, params }) {
    await store.dispatch('fetchArticles', { limit: 10, orderBy: 'date', exclude: store.state.featuredID })
    await store.dispatch('fetchFeaturedArticles', { limit: 10, orderBy: 'date' })
  },

  components: {
    ArticleList,
    Hero,
    InfiniteLoading,
    Sidebar
  },

  computed: {
    heroArticle () {
      return this.$store.state.articles[0]
    },
    indexInfiniteLoading () {
      return this.$store.state.indexInfiniteLoading
    }
  },

  head () {
    return {
      title: `Home | ${this.$store.state.meta.name}`,
      meta: [
        { description: this.$store.state.meta.description }
      ]
    }
  },

  methods: {
    moreArticles () {
      this.indexInfiniteLoading.page++

      this.$store.dispatch('fetchMoreArticles', { limit: 10, orderBy: 'date', exclude: this.$store.state.featuredID, page: this.indexInfiniteLoading.page })
        .then(() => {
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
.home {
  display: flex;

  .hero {
    margin: 0 -32px;
  }

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

<style lang="scss">
.home {
  .article-list {
    article {
      &:first-child {
        display: none;
      }

      &:nth-child(2) {
        border-top: 0;
        padding-top: 0;
      }
    }
  }
}
</style>

<template>
  <div class="home">
    <div class="articles">
      <TheHero
        v-if="heroArticle"
        :hero-article="heroArticle"
      />
      <ArticleList :articles="$store.state.articles" />
      <InfiniteLoading
        v-if="indexInfiniteLoading.enabled"
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
import InfiniteLoading from 'vue-infinite-loading/src/components/InfiniteLoading.vue'
import ArticleList from '~/components/ArticleList'
import TheHero from '~/components/TheHero'
import TheSidebar from '~/components/TheSidebar'
import Smile from '~/assets/svg/Smile.vue'
import Spinner1 from '~/components/Spinner1.vue'

export default {

  components: {
    ArticleList,
    TheHero,
    TheSidebar,
    InfiniteLoading,
    Smile,
    Spinner1
  },

  computed: {
    heroArticle () {
      return this.$store.state.articles[0]
    },
    indexInfiniteLoading () {
      return this.$store.state.indexInfiniteLoading
    }
  },
  async asyncData ({ app, store, params }) {
    if (!store.state.articles.length) {
      const articles = await app.$wp.posts(10).embed().excludeCategories(store.state.featuredID)
      store.commit('setArticles', articles)
    }

    if (!store.state.featuredArticles.length) {
      const articles = await app.$wp.posts(10).embed().categories(store.state.featuredID).embed()
      store.commit('setFeaturedArticles', articles)
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
    async moreArticles () {
      this.indexInfiniteLoading.page++

      try {
        const articles = await this.$wp.posts(10).excludeCategories(this.$store.state.featuredID).page(this.indexInfiniteLoading.page).embed()
        this.$store.commit('setArticles', articles)
        this.$refs.infiniteLoading.$emit('$InfiniteLoading:loaded')
      } catch (e) {
        this.$refs.infiniteLoading.$emit('$InfiniteLoading:complete')
      }
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

      @media (max-width: 700px) {
        margin: 16px 0;
      }
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

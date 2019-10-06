<template>
  <div class="page">
    <ul>
      <li v-for="topic in filteredTopics" :key="topic.id">
        <nuxt-link :to="`/topics/${topic.slug}`">
          <h2 v-html="topic.name" />
          <div v-html="topic.description" />
        </nuxt-link>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  computed: {
    filteredTopics () {
      return this.$store.state.topics.filter((topic) => {
        return topic.slug !== 'featured' && topic.count
      })
    }
  },
  async asyncData ({ app, store, params }) {
    if (!store.state.topics) {
      const topics = await app.$wp.categories(100)
      store.commit('setTopics', topics.data)
    }
  },

  head () {
    return {
      title: `Topics | ${this.$store.state.meta.name}`,
      meta: [
        { description: this.$store.state.meta.description }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>
@import '~assets/css/vars.scss';

.page {
  background-color: #efefef;
  padding: 32px;

  @media (max-width: 700px) {
    padding: 0 16px 32px 16px;
  }

  ul {
    column-count: 3;
    column-gap: 64px;
    margin: 0;
    padding: 0;
    list-style: none;

    @media (max-width: 700px) {
      column-count: 2;
    }

    @media (max-width: 500px) {
      column-count: 1;
    }

    li {
      display: flex;
      flex-direction: column;
      page-break-inside: avoid;
      break-inside: avoid;

      a {
        font-weight: 300;
      }

      & + li {
        margin-top: 32px;
      }

      h2 {
        display: inline-block;
        font-family: 'Roboto', sans-serif;
        font-size: 1.3rem;
        margin-bottom: 4px;
        margin-top: 0;
        font-weight: 400;
      }

      a {
        color: #111;

        &:hover div {
          color: #111;
        }

        div {
          color: lighten($primary, 10%);
          margin-top: 8px;
          transition: 0.1s;
        }
      }
    }
  }
}
</style>

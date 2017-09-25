<template>
  <div>
    <article v-for="article in articles" :key="article.id">
      <nuxt-link :to="`/${article.slug}`">
        <img :src="article._embedded['wp:featuredmedia'][0].media_details.sizes.thumbnail.source_url" v-if="article._embedded['wp:featuredmedia']">
        <h2 v-html="article.title.rendered"></h2>
        <div v-html="article.excerpt.rendered"></div>
      </nuxt-link>
    </article>
  </div>
</template>

<script>
export default {
  props: ['articles']
}
</script>

<style lang="scss" scoped>

</style>

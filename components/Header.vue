<template>
  <header v-if="meta">
    <nuxt-link to="/" exact>
      <h1 v-html="meta.name"></h1>
    </nuxt-link>
    <nav>
      <nuxt-link to="/" exact>Latest Articles</nuxt-link>
      <nuxt-link to="/topics">Topics</nuxt-link>
      <nuxt-link to="/authors">Authors</nuxt-link>
    </nav>
  </header>
</template>

<script>
export default {
  computed: {
    meta () {
      return this.$store.state.meta
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

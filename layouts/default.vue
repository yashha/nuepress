<template>
  <div>
    <BlogHeader/>
    <main role="main">
      <nuxt/>
    </main>
    <BlogFooter/>
  </div>
</template>

<script>
import BlogHeader from '../components/Header'
import BlogFooter from '../components/Footer'

export default {
  components: {
    BlogHeader,
    BlogFooter
  }
}
</script>

<style lang="scss">

</style>

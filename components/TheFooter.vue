<template>
  <footer>
    <div>
      <span class="fancy link" to="/" @click.prevent="homeScrollTop">
        <span v-html="meta.name" />
      </span>
      <div v-if="meta.description" v-html="meta.description" />
      <div>© {{ year }}</div>
    </div>
  </footer>
</template>

<script>
export default {

  mixins: {
    homeScrollTop: Function
  },
  computed: {
    meta () {
      return this.$store.state.meta
    },
    year () {
      return new Date().getFullYear()
    }
  }
}
</script>

<style lang="scss" scoped>
@import '~assets/css/vars.scss';

footer {
  > div {
    align-items: center;
    background-color: #fff;
    color: #aaa;
    display: flex;
    flex-direction: column;
    height: 200px;
    font-family: 'Roboto', sans-serif;
    font-size: .7rem;
    margin: 0 auto;
    max-width: $containerWidth;
    padding: 64px 32px;
    text-align: center;
  }

  .link {
    color: #aaa;
    font-weight: 400;
  }
}
</style>

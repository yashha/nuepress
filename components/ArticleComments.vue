<template>
  <div class="comments">
    <div v-if="!disqusReady" class="loading">
      <Spinner1 />
      <span>Loading comments</span>
    </div>
    <div class="disqus" :class="{ ready: disqusReady }">
      <client-only>
        <lazy-component>
          <VueDisqus
            shortname="nuepress-kmr-io"
            :identifier="article.slug"
            @ready="disqusReady = true"
          />
        </lazy-component>
      </client-only>
    </div>
  </div>
</template>

<script>
import VueDisqus from 'vue-disqus/dist/vue-disqus.vue'
import Spinner1 from '~/components/Spinner1'

export default {
  components: {
    Spinner1,
    VueDisqus
  },
  props: {
    article: {
      type: Object,
      default () {
        return {}
      }
    }
  },
  data () {
    return {
      disqusReady: false
    }
  }
}
</script>

<style lang="scss" scoped>
.comments {
  border-top: 1px dotted #65676a;
  padding-top: 32px;
  margin-top: 32px;

  .loading {
    align-items: center;
    display: flex;
    flex-direction: column;
    justify-content: center;
    position: absolute;
    width: 100%;

    .spinner {
      margin-bottom: 16px;
    }
  }

  .disqus {
    min-height: 440px;
    opacity: 0;
    transform: translateY(16px);
    transition: 1s;

    &.ready {
      opacity: 1;
      transform: translateY(0);
    }
  }
}
</style>

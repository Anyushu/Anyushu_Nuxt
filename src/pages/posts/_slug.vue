<template>
  <section class="Post">
    <PostContainer/>
  </section>
</template>

<script lang="ts">
import Vue from 'vue'
import PostContainer from '@/containers/Post/index.vue'

// Use Case
import FetchPostUseCase from '@/usecases/post/FetchPostUseCase'

// Repositories
import PostRepository from '@/repositories/PostRepository'

// Gateway
import ContentfulGateway from '@/gateway/ContentfulGateway'

// Service
import LogService from '@/services/LogService'

// Error
import { NotFoundError, ErrorType } from '@/common/errors'

export default Vue.extend({
  components: {
    PostContainer
  },
  head() {
    return {
      script: [
        {
          src: '//cdn.embedly.com/widgets/platform.js',
          async: true
        }
      ]
    }
  },
  async fetch({ params, store, $sentry, error }) {
    try {
      const usecase = new FetchPostUseCase({
        postRepository: new PostRepository(store),
        logService: new LogService({ logger: $sentry }),
        contentfulGateway: new ContentfulGateway()
      })
      await usecase.execute(params.slug)
    } catch (e) {
      if (e.type === ErrorType.NOT_FOUND) {
        error({ statusCode: 404, message: e.message })
        return
      }
      throw e
    }
  }
})
</script>


<style scoped>
.Post {
  width: 100%;
}
</style>
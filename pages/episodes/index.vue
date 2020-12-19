<template>
  <div class="container m-auto mt-8 page-transition px-2">
    <div class="text-center mb-6">
      <h3
        class="text-3xl leading-8 font-extrabold tracking-tight text-gray-900 sm:text-4xl sm:leading-10"
      >
        List of episodes
      </h3>
    </div>

    <episodes-table :episodes="episodes.results" />

    <client-only>
      <infinite-loading
        spinner="spiral"
        @infinite="infiniteScroll"
      ></infinite-loading>
    </client-only>
  </div>
</template>

<script lang="ts">
import { Context } from '@nuxt/types'
import { Component, Vue } from 'nuxt-property-decorator'
import gql from 'graphql-tag'

interface DataInterface {
  page: Number
  episodes: EpisodesInterface
}

interface EpisodesInterface {
  info: InfoPage
  results: Array<ResultData>
}

interface InfoPage {
  pages: number
}

interface ResultData {
  id: number
  name: string
  episode: string
  // eslint-disable-next-line camelcase
  air_date: string
  characters: Array<Character>
}

interface Character {
  id: number
  name: string
  image: string
}

const query = gql`
  query getEpisode($page: Int) {
    episodes(page: $page) {
      info {
        pages
      }
      results {
        id
        name
        episode
        air_date
        characters {
          id
          name
          image
        }
      }
    }
  }
`
@Component
export default class EpisodesPage extends Vue implements DataInterface {
  page: number = 1
  episodes!: EpisodesInterface

  async asyncData({ app }: Context): Promise<DataInterface> {
    const page = 1
    const result = await app?.apolloProvider?.defaultClient.query({
      query,
      variables: {
        page,
      },
    })

    const { episodes } = result?.data
    return {
      episodes,
      page,
    }
  }

  transition(
    to: { params: { page: string | number } },
    from: { params: { page: string | number } }
  ): string {
    if (!from) {
      return 'slide-left'
    }
    return +to.params.page < +from.params.page ? 'slide-right' : 'slide-left'
  }

  infiniteScroll($state: { loaded: () => void; complete: () => void }): void {
    setTimeout(async () => {
      this.page++
      if (this.page <= this.episodes.info.pages) {
        const result = await this.$apollo.query({
          query,
          variables: {
            page: this.page,
          },
        })

        const { results } = result.data.episodes
        if (results.length > 0) {
          results.forEach((item: ResultData) =>
            this.episodes.results.push(item)
          )
          $state.loaded()
        } else {
          $state.complete()
        }
      } else {
        $state.complete()
      }
    }, 500)
  }
}
</script>

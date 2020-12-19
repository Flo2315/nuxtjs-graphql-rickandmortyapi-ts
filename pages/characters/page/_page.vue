<template>
  <div class="container m-auto mt-8 page-transition px-2">
    <div class="text-center mb-6">
      <h3
        class="text-3xl leading-8 font-extrabold tracking-tight text-gray-900 sm:text-4xl sm:leading-10"
      >
        List of characters
      </h3>
    </div>
    <div class="grid grid-flow-row-dense md:grid-cols-3 grid-cols-2 gap-4">
      <template v-for="character in characters.results">
        <character-card :key="character.id" :character="character" />
      </template>
    </div>

    <pagination
      :info="characters.info"
      :characters-number-in-page="characters.results.length"
    />
  </div>
</template>

<script lang="ts">
import { Context } from '@nuxt/types'
import { Component, Vue } from 'nuxt-property-decorator'
import gql from 'graphql-tag'

@Component
export default class CharactersPage extends Vue {
  async asyncData({ app, params }: Context) {
    const result = await app?.apolloProvider?.defaultClient.query({
      query: gql`
        query getCharacters($page: Int) {
          characters(page: $page) {
            info {
              pages
              count
              next
              prev
            }
            results {
              id
              name
              image
              status
              species
              gender
              type
              created
              origin {
                name
                type
              }
              location {
                name
                type
              }
            }
          }
        }
      `,
      variables: {
        page: parseInt(params.page),
      },
    })

    return result?.data
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
}
</script>

<script lang="ts">
import { Context } from '@nuxt/types'
import { Component, Vue } from 'nuxt-property-decorator'

@Component
export default class CharactersPage extends Vue {
  fetch({ redirect }: Context) {
    redirect('/characters/page/1')
  }
}
</script>

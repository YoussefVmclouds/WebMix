<template>
  <v-app>
    <v-main>
      <div class="app-content-container boxed-container">
        <slot></slot>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import { ref } from '@vue/composition-api'
import { mdiMagnify, mdiBellOutline, mdiGithub } from '@mdi/js'

export default {
  components: {

  },
  setup() {
    const isDrawerOpen = ref(null)

    return {
      isDrawerOpen,

      // Icons
      icons: {
        mdiMagnify,
        mdiBellOutline,
        mdiGithub,
      },
    }
  },
}
</script>

<style lang="scss" scoped>
.v-app-bar ::v-deep {
  .v-toolbar__content {
    padding: 0;

    .app-bar-search {
      .v-input__slot {
        padding-left: 18px;
      }
    }
  }
}
.boxed-container {
  max-width: 100%;
  margin-left: 2.5vw;
  margin-right: 2.5vw;
  margin-top: 2.5vh;
  margin-bottom: 2.5vh;
}
</style>

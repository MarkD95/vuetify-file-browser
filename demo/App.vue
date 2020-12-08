<template>
  <v-app>
    <v-app-bar app color="secondary" dark>
      <div class="d-flex align-center">
        <h1>Vuetify File Browser</h1>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
        <FileBrowser />
    </v-main>
  </v-app>
</template>

<script>
import Vue from 'vue';

export default Vue.extend({
  name: 'App',
  components: {
    FileBrowser: () => import('./Browser'),
  },
});
</script>

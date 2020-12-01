<template>
  <v-app>
    <v-app-bar app color="secondary" dark>
      <div class="d-flex align-center">
        <h1>Vuetify File Browser</h1>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <v-row no-gutters>
        <FileBrowser
            :axiosConfig="{baseURL: 'http://localhost:8081'}"
        />
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
import Vue from 'vue';
import FileBrowser from '../src/FileBrowser.vue';

export default Vue.extend({
  name: 'App',
  components: {
    FileBrowser,
  },
  data () {
    return {
      name: '',
      url: 'http://',
    };
  },
  methods: {
  },
});
</script>

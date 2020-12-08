<template>
  <v-app>
    <v-app-bar app color="secondary" dark>
      <div class="d-flex align-center">
        <h1>Vuetify File Browser</h1>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <v-row no-gutters class="row-pad">
        <FileBrowser class="fbrowser"
            :axiosConfig="{baseURL: url}"
            :maxUploadFileSize="5242880"
            :maxUploadFilesCount="5"
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
      url: 'https://vuetify-file-browser-server.herokuapp.com',
    };
  },
});
</script>

<style>
.row-pad {
    padding: 1rem 0;
}
.fbrowser {
    width: 70%;
}
</style>

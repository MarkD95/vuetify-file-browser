<template>
    <v-row no-gutters class="row-pad">
        <FileBrowser class="fbrowser"
            :axiosConfig="{baseURL: url}"
            :maxUploadFileSize="5242880"
            :maxUploadFilesCount="5"
        />
    </v-row>
</template>

<script>
import Vue from 'vue';
import FileBrowser from '@/FileBrowser';
import '@mdi/font/css/materialdesignicons.css';

export default Vue.extend({
    name: 'BrowserDemo',
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

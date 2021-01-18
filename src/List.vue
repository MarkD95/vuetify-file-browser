<template>
    <v-card flat tile min-height="380" class="d-flex flex-column">
        <confirm ref="confirm"></confirm>
        <v-card-text
            v-if="!path"
            class="grow d-flex justify-center align-center grey--text"
        >Select a folder or a file</v-card-text>
        <v-card-text
            v-else-if="isFile"
            class="grow d-flex justify-center align-center"
        >File: {{ path }}</v-card-text>
        <v-card-text v-else-if="dirs.length || files.length" class="grow">
            <v-list subheader v-if="dirs.length">
                <v-subheader>Folders</v-subheader>
                <v-list-item
                    v-for="item in dirs"
                    :key="item.basename"
                    @click="changePath(item.path)"
                    @contextmenu.prevent="showFolder($event, item)"
                    class="pl-0"
                >
                    <v-list-item-avatar class="ma-0">
                        <v-icon>mdi-folder-outline</v-icon>
                    </v-list-item-avatar>
                    <v-list-item-content class="py-2">
                        <v-list-item-title v-text="item.basename"></v-list-item-title>
                    </v-list-item-content>
                    <v-list-item-action>
                        <v-btn icon @click.stop="deleteItem(item)">
                            <v-icon color="grey lighten-1">mdi-delete-outline</v-icon>
                        </v-btn>
                        <v-btn icon v-if="false">
                            <v-icon color="grey lighten-1">mdi-information</v-icon>
                        </v-btn>
                    </v-list-item-action>
                </v-list-item>
                <v-menu
                  v-model="showFolderMenu"
                  :position-x="x"
                  :position-y="y"
                  absolute
                  offset-y
                  style="max-width: 600px"
                >
                  <v-list>
                    <v-list-item>
                      <v-list-item-subtitle v-if="!!itemConcerned">{{itemConcerned.name}}</v-list-item-subtitle>
                    </v-list-item>
                    <v-list-item
                      v-for="(item, index) in $props.folderActions"
                      :key="index"
                      @click="item.action(itemConcerned)"
                    >
                      <v-list-item-avatar><v-icon :color="item.color ? item.color : ''">{{item.icon}}</v-icon></v-list-item-avatar>
                      <v-list-item-title>{{ item.title }}</v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>
            </v-list>
            <v-divider v-if="dirs.length && files.length"></v-divider>
            <v-list subheader v-if="files.length">
                <v-subheader>Files</v-subheader>
                  <v-list-item
                      v-for="item in files"
                      :key="item.basename"
                      @contextmenu.prevent="showFile($event, item)"
                      class="pl-0"
                  >
                      <v-list-item-avatar class="ma-0">
                          <v-icon>{{ icons[item.extension.toLowerCase()] || icons['other'] }}</v-icon>
                      </v-list-item-avatar>

                      <v-list-item-content class="py-2">
                          <v-list-item-title v-text="item.basename"></v-list-item-title>
                          <v-list-item-subtitle>{{ formatBytes(item.size) }}</v-list-item-subtitle>
                      </v-list-item-content>

                      <v-list-item-action>
                          <v-btn icon @click.stop="deleteItem(item)">
                              <v-icon color="grey lighten-1">mdi-delete-outline</v-icon>
                          </v-btn>
                          <v-btn icon v-if="false">
                              <v-icon color="grey lighten-1">mdi-information</v-icon>
                          </v-btn>
                      </v-list-item-action>
                  </v-list-item>
            </v-list>
            <v-menu
              v-model="showFileMenu"
              :position-x="x"
              :position-y="y"
              absolute
              offset-y
              style="max-width: 600px"
            >
              <v-list>
                <v-list-item>
                  <v-list-item-subtitle v-if="!!itemConcerned">{{itemConcerned.name}}</v-list-item-subtitle>
                </v-list-item>
                <v-list-item
                  v-for="(item, index) in $props.fileActions"
                  :key="index+item.title"
                  @click="item.action(itemConcerned)"
                >
                  <v-list-item-avatar><v-icon :color="item.color ? item.color : ''">{{item.icon}}</v-icon></v-list-item-avatar>
                  <v-list-item-title>{{ item.title }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
        </v-card-text>
        <v-card-text
            v-else-if="filter"
            class="grow d-flex justify-center align-center black--text py-5"
        >No files or folders found</v-card-text>
        <v-card-text
            v-else
            class="grow d-flex justify-center align-center black--text py-5"
        >The folder is empty</v-card-text>
        <v-divider v-if="path"></v-divider>
        <v-toolbar v-if="false && path && isFile" dense flat class="shrink">
            <v-btn icon>
                <v-icon>mdi-download</v-icon>
            </v-btn>
        </v-toolbar>
        <v-toolbar v-if="path && isDir" dense flat class="shrink">
            <v-text-field
                solo
                flat
                hide-details
                label="Filter"
                v-model="filter"
                prepend-inner-icon="mdi-filter-outline"
                class="ml-n3"
            ></v-text-field>
            <v-btn icon v-if="false" aria-label="Settings">
                <v-icon>mdi-eye-settings-outline</v-icon>
            </v-btn>
            <v-btn icon @click="$emit('reload')" aria-label="Refresh">
                <v-icon>mdi-refresh</v-icon>
            </v-btn>
        </v-toolbar>
    </v-card>
</template>

<script>
import { formatBytes } from './util';
import Confirm from './Confirm.vue';

export default {
    props: {
        icons: Object,
        storage: String,
        path: String,
        endpoints: Object,
        axios: Function,
        refreshPending: Boolean,
        items: Array,
        showFolderActions: Boolean,
        folderActions: Array,
        showFileActions: Boolean,
        fileActions: Array,
    },
    components: {
        Confirm
    },
    data() {
        return {
            filter: '',
            showFolderMenu: false,
            showFileMenu: false,
            x: 0,
            y: 0,
            itemConcerned: null
        };
    },
    computed: {
        dirs() {
            return this.items.filter(
                item =>
                    item.type === 'dir' && item.basename.includes(this.filter)
            );
        },
        files() {
            return this.items.filter(
                item =>
                    item.type === 'file' && item.basename.includes(this.filter)
            );
        },
        isDir() {
            return this.path[this.path.length - 1] === '/';
        },
        isFile() {
            return !this.isDir;
        }
    },
    methods: {
        formatBytes,
        changePath(path) {
            this.$emit('path-changed', path);
        },
        async deleteItem(item) {
            let confirmed = await this.$refs.confirm.open(
                'Delete',
                `Are you sure<br>you want to delete this ${
                    item.type === 'dir' ? 'folder' : 'file'
                }?<br><em>${item.basename}</em>`
            );

            if (confirmed) {
                this.$emit('loading', true);
                let url = this.endpoints.delete.url
                    .replace(new RegExp('{storage}', 'g'), this.storage)
                    .replace(new RegExp('{path}', 'g'), item.path);

                let config = {
                    url,
                    method: this.endpoints.delete.method || 'post'
                };

                await this.axios.request(config);
                this.$emit('file-deleted');
                this.$emit('loading', false);
            }
        },
        showFolder (e, item) {
          e.preventDefault()
          if (this.$props.showFolderActions == false) {
            return
          }
          this.showFolderMenu = false
          this.itemConcerned = item
          this.x = e.clientX
          this.y = e.clientY
          this.$nextTick(() => {
            this.showFolderMenu = true
          })
        },
        showFile (e, item) {
          e.preventDefault()
          if (this.$props.showFileActions == false) {
            return
          }
          this.showFileMenu = false
          this.itemConcerned = item
          this.x = e.clientX
          this.y = e.clientY
          this.$nextTick(() => {
            this.showFileMenu = true
          })
        },

    },
};
</script>

<style scoped>
.v-card {
    height: 100%;
}
</style>

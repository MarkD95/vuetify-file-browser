<template>
    <v-card class="mx-auto" :loading="loading > 0">
        <toolbar
            :path="path"
            :storages="storagesArray"
            :storage="activeStorage"
            :endpoints="endpoints"
            :axios="axiosInstance"
            v-on:storage-changed="storageChanged"
            v-on:path-changed="pathChanged"
            v-on:add-files="addUploadingFiles"
            v-on:folder-created="refreshPending = true"
        ></toolbar>
        <v-row no-gutters>
            <v-col v-if="tree && $vuetify.breakpoint.smAndUp" sm="auto">
                <tree
                    :path="path"
                    :storage="activeStorage"
                    :icons="icons"
                    :endpoints="endpoints"
                    :axios="axiosInstance"
                    :refreshPending="refreshPending"
                    v-on:path-changed="pathChanged"
                    v-on:loading="loadingChanged"
                    v-on:items="passItems"
                    v-on:refreshed="refreshPending = false"
                    v-on:reload="refreshPending = true"
                ></tree>
            </v-col>
            <v-divider v-if="tree" vertical></v-divider>
            <v-col>
                <list
                    :path="path"
                    :storage="activeStorage"
                    :showFolderActions="$props.showFolderActions"
                    :showFileActions="$props.showFileActions"
                    :folderActions="$props.folderActions"
                    :fileActions="$props.fileActions"
                    :icons="icons"
                    :endpoints="endpoints"
                    :items="items"
                    :axios="axiosInstance"
                    :refreshPending="refreshPending"
                    v-on:path-changed="pathChanged"
                    v-on:loading="loadingChanged"
                    v-on:refreshed="refreshPending = false"
                    v-on:file-deleted="refreshPending = true"
                    v-on:action="$emit('action', $event)"
                ></list>
            </v-col>
        </v-row>
        <upload
            v-if="uploadingFiles !== false"
            :path="path"
            :storage="activeStorage"
            :files="uploadingFiles"
            :icons="icons"
            :axios="axiosInstance"
            :endpoint="endpoints.upload"
            :maxUploadFilesCount="maxUploadFilesCount"
            :maxUploadFileSize="maxUploadFileSize"
            v-on:add-files="addUploadingFiles"
            v-on:remove-file="removeUploadingFile"
            v-on:clear-files="uploadingFiles = []"
            v-on:cancel="uploadingFiles = false"
            v-on:uploaded="uploaded"
        ></upload>
    </v-card>
</template>

<script>
import axios from 'axios';

import Toolbar from './Toolbar.vue';
import Tree from './Tree.vue';
import List from './List.vue';
import Upload from './Upload.vue';

const endpoints = {
    list: { url: '/storage/{storage}/list?path={path}', method: 'get' },
    upload: { url: '/storage/{storage}/upload?path={path}', method: 'post' },
    mkdir: { url: '/storage/{storage}/mkdir?path={path}', method: 'post' },
    delete: { url: '/storage/{storage}/delete?path={path}', method: 'post' }
};

const fileIcons = {
    zip: 'mdi-folder-zip-outline',
    rar: 'mdi-folder-zip-outline',
    htm: 'mdi-language-html5',
    html: 'mdi-language-html5',
    js: 'mdi-nodejs',
    json: 'mdi-json',
    md: 'mdi-markdown',
    pdf: 'mdi-file-pdf',
    png: 'mdi-file-image',
    jpg: 'mdi-file-image',
    jpeg: 'mdi-file-image',
    mp4: 'mdi-filmstrip',
    mkv: 'mdi-filmstrip',
    avi: 'mdi-filmstrip',
    wmv: 'mdi-filmstrip',
    mov: 'mdi-filmstrip',
    txt: 'mdi-file-document-outline',
    xls: 'mdi-file-excel',
    other: 'mdi-file-outline',
};

export default {
    components: {
        Toolbar,
        Tree,
        List,
        Upload,
    },
    model: {
        prop: 'path',
        event: 'change'
    },
    props: {
        // Available Storages array of objects
        availableStorages: {
            type: Array,
            validator: function (value) {
                function isObject(val) {
                    return Object.prototype.toString.call(val) == '[object Object]'
                }
                for (let i = 0; i < value.length; i++) {
                    const s = value[i];
                    if (!isObject(s)) {
                        return false;
                    }
                    if (s['name'] == undefined || s['code'] == undefined || s['icon'] == undefined) {
                        return false;
                    }
                }
                // The value must match one of these strings
                return true;
            },
            default: () => {return [
                {
                    name: 'Local',
                    code: 'local',
                    icon: 'mdi-folder-multiple-outline'
                },
                {
                    name: 'Amazon S3',
                    code: 's3',
                    icon: 'mdi-amazon-drive'
                }
                /*{
                    name: 'Dropbox',
                    code: 'dropbox',
                    icon: 'mdi-dropbox'
                }*/
            ]}
        },
        showFolderActions: {
            type: Boolean,
            default: false,
        },
        folderActions: {
            type: Array,
            default: () => {return [
                {title: 'View', icon: 'mdi-eye', action: function (item) {this.$emit('action', {item, type: 'folder', actionType: 'view'})}},
                {title: 'Copy', icon: 'mdi-file-document-multiple', action: function (item) {this.$emit('action', {item, type: 'folder', actionType: 'copy'})}},
                {title: 'Move', icon: 'mdi-file-move', action: function (item) {this.$emit('action', {item, type: 'folder', actionType: 'move'})}},
                {title: 'Delete', icon: 'mdi-file-remove', color: 'error', action: function (item) {this.$emit('action', {item, type: 'folder', actionType: 'delete'})}},
            ]}
        },
        showFileActions: {
            type: Boolean,
            default: false,
        },
        fileActions: {
            type: Array,
            default: () => {return [
                {title: 'View', icon: 'mdi-eye', action: function (item) {this.$emit('action', {item, type: 'file', actionType: 'view'})}},
                {title: 'Copy', icon: 'mdi-file-document-multiple', action: function (item) {this.$emit('action', {item, type: 'file', actionType: 'copy'})}},
                {title: 'Move', icon: 'mdi-file-move', action: function (item) {this.$emit('action', {item, type: 'file', actionType: 'move'})}},
                {title: 'Delete', icon: 'mdi-file-remove', color: 'error', action: function (item) {this.$emit('action', {item, type: 'file', actionType: 'delete'})}},
            ]}
        },
        // comma-separated list of active storage codes
        storages: {
            type: String,
            default() {
                return this.$props.availableStorages.map(item => item.code).join(',')
            }
        },
        // code of default storage
        storage: { type: String, default: 'local' },
        // show tree view
        tree: { type: Boolean, default: true },
        // file icons set
        icons: { type: Object, default: () => fileIcons },
        // custom backend endpoints
        endpoints: { type: Object, default: () => endpoints },
        // custom axios instance
        axios: { type: Function },
        // custom configuration for internal axios instance
        axiosConfig: { type: Object, default: () => {} },
        // max files count to upload at once. Unlimited by default
        maxUploadFilesCount: { type: Number, default: 0 },
        // max file size to upload. Unlimited by default
        maxUploadFileSize: { type: Number, default: 0 }
    },
    data() {
        return {
            loading: 0,
            path: '',
            activeStorage: null,
            uploadingFiles: false, // or an Array of files
            refreshPending: false,
            axiosInstance: null,
            items: [],
        };
    },
    computed: {
        storagesArray() {
            let storageCodes = this.storages.split(','),
                result = [];
            storageCodes.forEach(code => {
                result.push(this.$props.availableStorages.find(item => item.code == code));
            });
            return result;
        }
    },
    methods: {
        loadingChanged(loading) {
            if (loading) {
                this.loading++;
            } else if (this.loading > 0) {
                this.loading--;
            }
        },
        storageChanged(storage) {
            this.activeStorage = storage;
        },
        addUploadingFiles(files) {
            files = Array.from(files);

            if (this.maxUploadFileSize) {
                files = files.filter(
                    file => file.size <= this.maxUploadFileSize
                );
            }

            if (this.uploadingFiles === false) {
                this.uploadingFiles = [];
            }

            if (this.maxUploadFilesCount && this.uploadingFiles.length + files.length > this.maxUploadFilesCount) {
                files = files.slice(0, this.maxUploadFilesCount - this.uploadingFiles.length);
            }

            this.uploadingFiles.push(...files);
        },
        removeUploadingFile(index) {
            this.uploadingFiles.splice(index, 1);
        },
        uploaded() {
            this.uploadingFiles = false;
            this.refreshPending = true;
        },
        pathChanged(path) {
            this.path = path;
            this.items = [];
            this.$emit('change', path);
            this.refreshPending = true;
        },
        passItems(items) {
            this.items = items;
        }
    },
    created() {
        this.activeStorage = this.storage;
        this.axiosInstance = this.axios || axios.create(this.axiosConfig);
    },
    mounted() {
        if (!this.path && !(this.tree && this.$vuetify.breakpoint.smAndUp)) {
            this.pathChanged('/');
        }
    }
};
</script>

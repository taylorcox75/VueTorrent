<template>
    <v-card
        class="mt-5 mx-5"
        flat
        v-if="fileTree"
        style="position: relevative; max-height: 500px; min-height: 400px;"
    >
        <treeselect
            :options="fileTree"
            :multiple="true"
            :flatten-search-results="true"
            :show-count="true"
            v-model="value"
        />
        <v-card-actions
            @click="exclude"
            text
            style="position: absolute; bottom: 14px; left: 40%; right: 50%;"
        >
            <v-btn
                @click="exclude"
                text
                class="blue_accent white--text mx-0 mt-3"
                >Exclude</v-btn
            >
        </v-card-actions>
    </v-card>
</template>

<script>
/* eslint-disableno-unused-vars */
// import the component
import Treeselect from '@riophae/vue-treeselect'
// import the styles
import '@riophae/vue-treeselect/dist/vue-treeselect.css'

import qbit from '@/services/qbit'
import { formatBytes } from '@/helpers'

function treeifySelect(paths) {
    let result = []
    let level = { result }

    paths.forEach(path => {
        path.name.split('/').reduce((r, name) => {
            if (!r[name]) {
                r[name] = { result: [] }
                r.result.push({ name, children: r[name].result, item: path })
            }

            return r[name]
        }, level)
    })

    console.log(result)

    //parse folders
    result = result.map(el => parseResult(el))

    function parseResult(el) {
        if (el.children.length !== 0) {
            let folder = createFolder(el)
            folder.children = folder.children.map(el => parseResult(el))
            return folder
        }
        return createFile(el)
    }
    
    console.log(result)

    return result
}

function getIconForFileType(type) {
    let types = {
        avi: '🎞',
        mp4: '🎞',
        mkv: '🎞'
    }

    if (!types[type]) return '📄'

    return types[type]
}

function createFile(el) {
    return {
        id: el.name,
        item: el,
        label: `${getIconForFileType(el.name.split('.').pop())} ${
            el.name
        } [${formatBytes(el.item.size)}] ${Math.round(el.item.progress * 100)}%`
    }
}

function createFolder(el) {
    return {
        id: el.name,
        children: el.children,
        label: `📁 ${el.name}`,
        item: el
    }
}

export default {
    name: 'Exclude',
    components: { Treeselect },
    props: {
        hash: String,
        isActive: Boolean
    },
    data: () => ({
        value: null,
        tree: [],
        treeData: null
    }),
    computed: {
        fileTree() {
            if (this.treeData) {
                const treeified = treeifySelect(this.treeData)
                return treeified
            }
            return []
        }
    },
    methods: {
        async getTorrentFiles() {
            const { data } = await qbit.getTorrentFiles(this.hash)
            this.treeData = data
        },
        exclude() {
            console.log(this.value)
        }
    },
    watch: {
        isActive(active) {
            if (active) {
                this.getTorrentFiles()
            } else {
                this.value = null
            }
        }
    }
}
</script>

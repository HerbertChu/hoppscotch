<template>
  <div>
    <div class="flex-wrap">
      <button class="icon" @click="toggleShowChildren">
        <i class="material-icons" v-show="!showChildren">arrow_right</i>
        <i class="material-icons" v-show="showChildren">arrow_drop_down</i>
        <folderIcon class="material-icons" />
        <span>{{ collection.name }}</span>
      </button>
      <div>
        <button
          v-if="doc"
          class="icon"
          @click="$emit('select-collection')"
          v-tooltip.left="$t('import')"
        >
          <i class="material-icons">topic</i>
        </button>
        <v-popover>
          <button class="tooltip-target icon" v-tooltip.left="$t('more')">
            <i class="material-icons">more_vert</i>
          </button>
          <template slot="popover">
            <div>
              <button class="icon" @click="$emit('add-folder')" v-close-popover>
                <i class="material-icons">create_new_folder</i>
                <span>{{ $t("new_folder") }}</span>
              </button>
            </div>
            <div>
              <button class="icon" @click="$emit('edit-collection')" v-close-popover>
                <i class="material-icons">create</i>
                <span>{{ $t("edit") }}</span>
              </button>
            </div>
            <div>
              <button class="icon" @click="removeCollection" v-close-popover>
                <deleteIcon class="material-icons" />
                <span>{{ $t("delete") }}</span>
              </button>
            </div>
          </template>
        </v-popover>
      </div>
    </div>

    <div v-show="showChildren">
      <ul>
        <li v-for="(folder, index) in collection.folders" :key="folder.name">
          <folder
            :folder="folder"
            :folderIndex="index"
            :collection-index="collectionIndex"
            :doc="doc"
            @edit-folder="editFolder(collectionIndex, folder, index)"
            @edit-request="$emit('edit-request', $event)"
          />
        </li>
        <li v-if="collection.folders.length === 0 && collection.requests.length === 0">
          <label>{{ $t("collection_empty") }}</label>
        </li>
      </ul>
      <ul>
        <li v-for="(request, index) in collection.requests" :key="index">
          <request
            :request="request"
            :collection-index="collectionIndex"
            :folder-index="-1"
            :request-index="index"
            :doc="doc"
            @edit-request="
              $emit('edit-request', {
                request,
                collectionIndex,
                folderIndex: undefined,
                requestIndex: index,
              })
            "
          />
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped lang="scss">
ul {
  display: flex;
  flex-direction: column;
}

ul li {
  display: flex;
  margin-left: 32px;
  border-left: 1px solid var(--brd-color);
}
</style>

<script>
import { fb } from "~/helpers/fb"
import folderIcon from "~/static/icons/folder-24px.svg?inline"
import deleteIcon from "~/static/icons/delete-24px.svg?inline"

export default {
  components: { folderIcon, deleteIcon },
  props: {
    collectionIndex: Number,
    collection: Object,
    doc: Boolean,
  },
  data() {
    return {
      showChildren: false,
      selectedFolder: {},
    }
  },
  methods: {
    syncCollections() {
      if (fb.currentUser !== null) {
        if (fb.currentSettings[0].value) {
          fb.writeCollections(JSON.parse(JSON.stringify(this.$store.state.postwoman.collections)))
        }
      }
    },
    toggleShowChildren() {
      this.showChildren = !this.showChildren
    },
    removeCollection() {
      if (!confirm(this.$t("are_you_sure_remove_collection"))) return
      this.$store.commit("postwoman/removeCollection", {
        collectionIndex: this.collectionIndex,
      })
      this.$toast.error(this.$t("deleted"), {
        icon: "delete",
      })
      this.syncCollections()
    },
    editFolder(collectionIndex, folder, folderIndex) {
      this.$emit("edit-folder", { collectionIndex, folder, folderIndex })
    },
  },
}
</script>

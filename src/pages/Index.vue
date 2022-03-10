<template>
  <q-page class="column">
    <div class="q-pa-md q-gutter-lg fit row justify-end">
      <q-checkbox
        label="Expand/Collapse all"
        v-model="expandTree"
        @click="expandOrCollapseTree"
      />
      <q-btn side @click="triggerClick" label="Update tree" />
    </div>

    <div class="q-pa-md q-gutter-sm">
      <q-input ref="filterRef" filled v-model="filter" label="Search">
        <template v-slot:append>
          <q-icon
            v-if="filter !== ''"
            name="clear"
            class="cursor-pointer"
            @click="resetFilter"
          />
        </template>
      </q-input>

      <q-tree
        ref="nodes"
        :nodes="bookmarksTree"
        node-key="id"
        :filter="filter"
        :filter-method="myFilterMethod"
        default-expand-all
      >
        <template v-slot:default-header="prop">
          <div class="row items-center">
            <img :src="prop.node.avatar" class="q-mr-sm" />
            <a @click="openInNewTab(prop.node.url)">
              {{ prop.node.title }}
            </a>
          </div>
        </template>
      </q-tree>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import { ref } from "vue";

export default defineComponent({
  name: "PageIndex",
  setup() {
    const filter = ref("");
    const filterRef = ref(null);

    return {
      filter,
      filterRef,

      myFilterMethod(node, filter) {
        const filt = filter.toLowerCase();
        return node.title && node.title.toLowerCase().indexOf(filt) > -1;
      },

      resetFilter() {
        filter.value = "";
        filterRef.value.focus();
      },
    };
  },
  mounted() {
    this.updateTree();
    window.setInterval(this.updateTree, 5000);
  },
  unmounted() {
    window.clearInterval();
  },
  data() {
    return {
      bookmarksTree: [],
      expandTree: false,
    };
  },
  methods: {
    updateTree() {
      chrome.bookmarks.getTree((bookmarkTreeNode) => {
        this.bookmarksTree = bookmarkTreeNode;
        console.log("UPDATING TREE");
        this.traverseDFS(this.bookmarksTree);
        console.log(this.bookmarksTree);
      });
    },
    traverseDFS(root) {
      //if there is no root, return false
      if (!root[0]) {
        return false;
      }

      //current values always starts at root
      let current = root[0];

      this.preOrderHelper(current);
      console.log("UPDATED: ", this.bookmarksTree);
    },
    preOrderHelper(node) {
      // Fix issue where title can be empty
      if (
        node.title != undefined &&
        node.title.length == 0 &&
        node.url != undefined
      ) {
        node.title = node.url;
      }

      if (node.url) {
        let domain = new URL(node.url);
        domain = domain.hostname;
        node["avatar"] = "http://www.google.com/s2/favicons?domain=" + domain;
        console.log(node.avatar);
      } else {
        node["avatar"] =
          "https://img.icons8.com/color/24/000000/folder-invoices--v1.png";
      }

      // node["icon"] = !node.url ? "folder" : "";
      //recursively call function on all node children
      if (node.children && node.children.length !== 0) {
        node.children.forEach((child) => {
          this.preOrderHelper(child);
        });
      }
      return true;
    },
    triggerClick() {
      console.log("Triggered");
      this.updateTree();
    },
    expandOrCollapseTree() {
      this.expandTree
        ? this.$refs.nodes.expandAll()
        : this.$refs.nodes.collapseAll();
    },
    openInNewTab(url) {
      if (url == undefined || url.length === 0) {
        return;
      }
      window.open(url, "_blank").focus();
    },
  },
  updated() {
    expandOrCollapseTree();
  },
});
</script>

<style lang="scss">
a,
a label {
  cursor: pointer;
}
</style>

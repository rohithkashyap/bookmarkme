<template>
  <q-page class="col q-pa-md">
    <p>Reads bookmarks and shows in a tree layout</p>
    <p>Bookmarks are refreshed every 5 seconds</p>
    <hr />
    <p class="text-h5">Changelog</p>
    <div v-for="updates in changelog" :key="updates.version">
      <p class="text-h6">{{ updates.version }}</p>
      <li v-for="change in updates.changes" :key="change">
        {{ change }}
      </li>
      <br />
    </div>
    <p></p>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "PageIndex",
  data() {
    return {
      changelog: [
        {
          version: "0.0.2",
          changes: [
            "Use URL for bookmarks that don't have titles",
            "Add icons for folders and favicons for websites",
          ],
        },
        {
          version: "0.0.1",
          changes: ["Base app"],
        },
      ],
    };
  },
});
</script>

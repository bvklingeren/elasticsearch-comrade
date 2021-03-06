<template>
  <v-layout>
    <v-flex xs7 class="pr-4">
      <v-header sub>Index Templates</v-header>
      <v-data-table
        :headers="headers"
        :items="templates"
        class="elevation-1 small-table"
        item-key="name"
        :footer-props="{ 'items-per-page-options': [30, 100, 200] }"
      >
        <template v-slot:item.actions="{ item }">
          <v-btn icon small @click="edit(item)">
            <v-icon small>edit</v-icon>
          </v-btn>
          <v-btn icon small @click="deleteT(item.name)">
            <v-icon small>delete_outline</v-icon>
          </v-btn>
        </template>
        <template v-slot:item.index_patterns="{ item }">
          {{ item.index_patterns.join(", ") }}
        </template>
      </v-data-table>
    </v-flex>
    <v-flex xs5 class="pl-4">
      <v-header sub>{{ isNew ? "Create" : "Edit" }} Template</v-header>
      <v-text-field label="Template name*" v-model="templateName" />
      <query-editor style="height: 500px" :init="base" ref="editor" />
      <v-btn
        block
        class="mt-6"
        color="success"
        :disabled="!templateName"
        @click="submit"
      >
        {{ isNew ? "Create Template" : "Update Template" }}
      </v-btn>
    </v-flex>
  </v-layout>
</template>

<script>
import templateApis from "../mixins/templateApis";
import VHeader from "../components/Base/Header.vue";
import QueryEditor from "../components/Base/QueryEditor.vue";
import { mapMutations } from "vuex";

export default {
  mixins: [templateApis],
  components: { VHeader, QueryEditor },
  async created() {
    this.updateTable();
  },
  data() {
    return {
      templates: [],
      templateName: "",
      headers: [
        { text: "Name", value: "name" },
        { text: "Index Pattern", value: "index_patterns" },
        { text: "Actions", value: "actions", sortable: false }
      ],
      base: {
        index_patterns: ["te*", "bar*"],
        settings: {
          number_of_shards: 1
        },
        mappings: {}
      }
    };
  },
  methods: {
    ...mapMutations(["startLoading", "stopLoading"]),
    edit(template) {
      this.templateName = template.name;
      this.$refs.editor.setContent(
        Object.assign({}, template, { name: undefined })
      );
    },
    async updateTable() {
      this.templates = await this.listTemplates();
    },
    async submit() {
      this.startLoading();
      const body = this.$refs.editor.getQuery();
      if (this.isNew) {
        await this.createTemplate(this.templateName, body);
      } else {
        await this.updateTemplate(this.templateName, body);
      }
      await this.updateTable();
      this.stopLoading();
    },
    async deleteT(templateName) {
      this.startLoading();
      await this.deleteTemplate(templateName);
      await this.updateTable();
      this.stopLoading();
    }
  },
  computed: {
    isNew() {
      return (
        this.templates.filter(x => x.name === this.templateName).length === 0
      );
    }
  }
};
</script>

<style scoped></style>

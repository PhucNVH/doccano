<template>
  <base-card
    :title="sds"
    :cancel-text="$t('generic.close')"
    @cancel="$emit('click:cancel')"
  >
    <v-dialog v-model="dialogComment">
      <form-create
        v-bind.sync="editedItem"
        :used-keys="usedKeys"
        :used-names="usedNames"
        @cancel="close"
        @save="save"
      />
    </v-dialog>
  </base-card>
</template>

<script lang="ts">
import Vue from "vue";
import BaseCard from "@/components/utils/BaseCard.vue";
import FormCreate from "@/components/label/FormCreate.vue";
import FormDelete from "@/components/label/FormDelete.vue";
import FormUpload from "@/components/label/FormUpload.vue";
import LabelList from "@/components/label/LabelList.vue";
import { CommentReadDTO } from "~/services/application/comment/commentData";
import { LabelDTO } from "~/services/application/label/labelData";

export default Vue.extend({
  components: {
    BaseCard,
    FormCreate,
    // FormDelete,
    // FormUpload,
    // LabelList
  },

  props: {
    docId: {
      type: Number,
      required: true,
    },
  },

  async fetch() {
    this.user = await this.$services.user.getMyProfile();
  },

  data() {
    return {
      user: {},
      dialogComment: false,
      dialogDelete: false,
      dialogUpload: false,
      editedIndex: -1,
      editedItem: {
        text: '',
        prefixKey: null,
        suffixKey: null,
        backgroundColor: '#2196F3',
        textColor: '#ffffff'
      } as LabelDTO,
      defaultItem: {
        text: '',
        prefixKey: null,
        suffixKey: null,
        backgroundColor: '#2196F3',
        textColor: '#ffffff'
      } as LabelDTO,
      items: [] as LabelDTO[],
      selected: [] as LabelDTO[],
      isLoading: false,
      errorMessage: ''
    }
  },
  computed: {
    canDelete(): boolean {
      return this.selected.length > 0
    },
    projectId(): string {
      return this.$route.params.id
    },
    usedNames(): string[] {
      const item = this.items[this.editedIndex] // to remove myself
      return this.items.filter(_ => _ !== item).map(item => item.text)
    },
    usedKeys(): string[] {
      const item = this.items[this.editedIndex] // to remove myself
      return this.items.filter(_ => _ !== item).map(item => item.suffixKey)
                       .filter(item => item !==null) as string[]
    }
  },

  watch: {
    currentDoc: {
      handler(val) {
        if (val !== undefined) {
          this.list();
        }
      },
      immediate: true,
      deep: true,
    },
  },
  methods: {
    async list() {
      // this.comments = await this.$services.comment.list(
      //   this.$route.params.id,
      //   this.docId
      // );
    },
    async add(message: string) {
      // await this.$services.comment.create(
      //   this.$route.params.id,
      //   this.docId,
      //   message
      // );
      // this.list();
    },
    async maybeUpdate(item: CommentReadDTO) {
      await this.$services.comment.update(
        this.$route.params.id,
        this.docId,
        item
      );
      this.list();
    },
    async create() {
      await this.$services.label.create(this.projectId, this.editedItem);
    },

    async update() {
      await this.$services.label.update(this.projectId, this.editedItem);
    },

    save() {
      if (this.editedIndex > -1) {
        this.update();
      } else {
        this.create();
      }
      this.$fetch();
      this.close();
    },
    async remove() {
      await this.$services.label.bulkDelete(this.projectId, this.selected);
      this.$fetch();
      this.dialogDelete = false;
      this.selected = [];
    },
    close() {
      this.dialogComment = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
  },
});
</script>

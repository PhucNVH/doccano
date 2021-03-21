<template>
  <v-toolbar
    class="toolbar-control"
    dense
    flat
  >
    <v-row no-gutters>
      <v-btn-toggle>
        <button-review
          v-if="showApproveButton"
          :is-reviewd="isReviewd"
          @click:review="$emit('click:review')"
        />

        <button-filter
          :value="filterOption"
          @click:filter="changeFilter"
        />

        <button-guideline
          @click:guideline="dialogGuideline=true"
        />
        <v-dialog v-model="dialogGuideline">
          <form-guideline
            :guideline-text="guidelineText"
            @click:close="dialogGuideline=false"
          />
        </v-dialog>

        <button-comment
          @click:comment="dialogComment=true"
        />
        <v-dialog v-model="dialogComment">
           <form-create
          v-bind.sync="editedItem"
          :used-keys="usedKeys"
          :used-names="usedNames"
          @cancel="close"
          @save="save"
        />
        </v-dialog>

        <button-auto-labeling
          @click:auto="dialogAutoLabeling=true"
        />
        <v-dialog v-model="dialogAutoLabeling">
          <form-auto-labeling
            :is-enabled="enableAutoLabeling"
            :error-message="errorMessage"
            @click:cancel="dialogAutoLabeling=false"
            @input="updateAutoLabeling"
          />
        </v-dialog>

        <button-clear
          @click:clear="dialogClear=true"
        />
        <v-dialog v-model="dialogClear">
          <form-clear-label
            @click:ok="$emit('click:clear-label');dialogClear=false"
            @click:cancel="dialogClear=false"
          />
        </v-dialog>
      </v-btn-toggle>
      <v-spacer />
      <button-pagination
        :value="page"
        :total="total"
        @click:prev="updatePage(page - 1)"
        @click:next="updatePage(page + 1)"
        @click:first="updatePage(1)"
        @click:last="updatePage(total)"
        @click:jump="updatePage($event)"
      />
    </v-row>
  </v-toolbar>
</template>

<script lang="ts">
import Vue from 'vue'
import LabelList from '@/components/label/LabelList.vue'
import ButtonAutoLabeling from './buttons/ButtonAutoLabeling.vue'
import ButtonClear from './buttons/ButtonClear.vue'
import ButtonComment from './buttons/ButtonComment.vue'
import ButtonFilter from './buttons/ButtonFilter.vue'
import ButtonGuideline from './buttons/ButtonGuideline.vue'
import ButtonPagination from './buttons/ButtonPagination.vue'
import ButtonReview from './buttons/ButtonReview.vue'
import FormAutoLabeling from './forms/FormAutoLabeling.vue'
import FormClearLabel from './forms/FormClearLabel.vue'
import FormCreate from './forms/FormCreate.vue'
import FormGuideline from './forms/FormGuideline.vue'
import { LabelDTO } from '~/services/application/label/labelData'

export default Vue.extend({
  components: {
    ButtonAutoLabeling,
    ButtonClear,
    ButtonComment,
    ButtonFilter,
    ButtonGuideline,
    ButtonPagination,
    ButtonReview,
    FormAutoLabeling,
    FormClearLabel,
    FormCreate,
    FormGuideline
  },

  props: {
    docId: {
      type: Number,
      required: true
    },
    enableAutoLabeling: {
      type: Boolean,
      default: false,
      required: true
    },
    guidelineText: {
      type: String,
      default: '',
      required: true
    },
    isReviewd: {
      type: Boolean,
      default: false
    },
    showApproveButton: {
      type: Boolean,
      default: false,
      required: true
    },
    total: {
      type: Number,
      default: 1
    }
  },
  async fetch() {
    this.items = await this.$services.label.list(this.projectId)
  },

  data() {
    return {
      dialogAutoLabeling: false,
      dialogClear: false,
      dialogComment: false,
      dialogGuideline: false,
      errorMessage: '',
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
    }
  },

  computed: {
    page(): number {
      // @ts-ignore
      return parseInt(this.$route.query.page, 10)
    },
    filterOption(): string {
      // @ts-ignore
      return this.$route.query.isChecked
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

  methods: {
    async create() {
      await this.$services.label.create(this.projectId, this.editedItem)
      this.$emit('label-updated','label-updated');
    },

    async update() {
      await this.$services.label.update(this.projectId, this.editedItem)
      
    },

    save() {
      if (this.editedIndex > -1) {
        this.update()
      } else {
        this.create()
      }
      this.$fetch()
      this.close()
    },

    close() {
      this.dialogComment = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },


    clearErrorMessage() {
      this.errorMessage = ''
    },

    editItem(item: LabelDTO) {
      this.editedIndex = this.items.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogComment = true
    },
    updatePage(page: number) {
      this.$router.push({ query: {
        page: page.toString(),
        isChecked: this.filterOption,
        q: this.$route.query.q
      }})
    },

    changeFilter(isChecked: string) {
      this.$router.push({ query: {
        page: '1',
        isChecked,
        q: this.$route.query.q
      }})
    },
    
    async updateAutoLabeling(isEnable: boolean) {
      if (isEnable) {
        try {
          const projectId = this.$route.params.id
          await this.$services.textClassification.autoLabel(projectId, this.docId)
          this.$emit('update:enable-auto-labeling', true)
          this.errorMessage = ''
        } catch (e) {
          this.errorMessage = e.response.data.detail
        }
      } else {
        this.$emit('update:enable-auto-labeling', false)
      }
    }
  }
})
</script>

<style scoped>
.toolbar-control >>> .v-toolbar__content {
  padding: 0px !important;
}

::v-deep .v-dialog {
  width: 800px;
}
</style>
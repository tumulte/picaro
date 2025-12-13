<script lang="ts" setup>
import ModelForm from "./ModelForm.vue"
import {computed, ref} from "vue";
import {Category, Model, ModelContent, Settings} from "@types";
import DisplayList from "@components/display/DisplayList.vue";
import FilterCategories from "@components/filters/FilterCategories.vue";
import {useSettingsStore} from "@stores/settings";
import {useRouter} from "vue-router";
import {nanoid} from "nanoid";
import {useUtilsStore} from "@stores/utils";
import {updateSettings} from "@components/utils/api";
import draggable from "vuedraggable";
import ImageUpload from "@components/ImageUpload.vue";

defineEmits(["updateModelFormState", "cancelEditModel"])

const props = defineProps<{
  currentEditModel: Model
}>()

const selectedStatus = ref<ModelContent['status']>('published')

const possibleStatus: ModelContent['status'][] = ['published', "draft", "archived", 'deleted']

const settingsStore = useSettingsStore();
const utilsStore = useUtilsStore();

const router = useRouter()

const dataReloaded = ref(false)


const categories = computed<Category[]>(() => settingsStore.currentAppSettings?.categories.filter(item => item.model === props.currentEditModel.id) || [])

const newCategory = ref('')
const editCategories = ref(false)

function editItem(item: number) {
  router.push({params: {contentId: item}}).catch(e => console.error(e))
  newCategory.value = ''
}

function addCategory() {
  settingsStore.currentAppSettings?.categories.push({
    label: newCategory.value,
    id: nanoid(6),
    model: props.currentEditModel.id
  })
  newCategory.value = ''
}

function cancelEdit() {
  window.location.reload()
}

function deleteCategory(index: number) {
  utilsStore.awaitConfirmation({
    text: "Are you sure you want to delete this category?",
    type: "warning"
  }).then(() => {
    settingsStore.currentAppSettings?.categories.splice(index, 1)
  }).catch((error) => console.error(error))
}

async function saveCategory() {
  await updateSettings(settingsStore.currentAppSettings as Settings)
  editCategories.value = false
}

</script>
<template>
  <VRow>
    <VCol :cols="3">
      <h3>
        Existing categories :
      </h3>
      <button
        v-if="!editCategories"
        class="pic-button--text pic-button--text__categories"
        data-testid="edit-categories"
        @click="editCategories = !editCategories"
      >
        Edit categories
      </button>
      <VBtn
        v-else
        color="secondary"
        variant="text"
        @click="cancelEdit()"
      >
        Cancel
      </VBtn>
      <FilterCategories
        v-if="settingsStore.currentAppSettings && !editCategories"
        :current-app="settingsStore.currentAppSettings"
        :current-model-id="currentEditModel.id"
      />
      <div v-else-if="settingsStore.currentAppSettings">
        <draggable
          v-model="settingsStore.currentAppSettings.categories"
          ghost-class="pic-sortable-ghost"
          handle=".pic-sortable-handle"
          item-key="id"
        >
          <template #item="{element,index}">
            <div v-if="element.model === currentEditModel.id">
              <VCheckbox
                v-model="element.section"
                data-testid="separator-check"
                density="compact"
                hide-details
                label="Separator (non-clickable)"
              />

              <div class="pic-flex">
                <span class="pic-sortable-handle pic-sortable-handle__category">
                  <v-icon>mdi-drag</v-icon>
                </span>
                <v-text-field
                  :key="element.id"
                  v-model="element.label"
                  append-icon="mdi-delete"
                  @click:append="deleteCategory(index)"
                />
              </div>
            </div>
          </template>
        </draggable>

        <VTextField v-model="newCategory" data-testid="new-category-input" label="New category"/>
        <VBtn data-testid="new-category-add" @click="addCategory">
          Add category
        </VBtn>
        <br>
        <VBtn class="mt-4" color="primary" data-testid="new-category-save" @click="saveCategory()">
          Save
        </VBtn>
      </div>
    </VCol>
    <VCol>
      <h3
        data-testid="new-content"
      >
        New content :
      </h3>
      <ModelForm
        :categories="categories"
        :current-edit-model="currentEditModel"
        data-testid="new-content-form"
        @reloadData="dataReloaded = !dataReloaded"
      />
      <div class="v-label mt-4">
        Edit existing content :
      </div>
      <div>
        <VBtn
          v-for="status in possibleStatus"
          :key="status"
          :color="status === selectedStatus ? 'primary' : ''"
          variant="text"
          @click="selectedStatus = status"
        >
          {{ status }}
        </VBtn>
      </div>
      <DisplayList
        v-if="settingsStore.currentAppSettings"
        :categories="categories"
        :current-app="settingsStore.currentAppSettings"
        :data-reloaded="dataReloaded"
        :display-all="true"
        :display-status="selectedStatus"
        :module-params="{model: currentEditModel.id, categories: [], type: 'List'}"
        class="pic-display-edit"
        data-testid="content-list"
        @clickItem="editItem($event)"
      />
    </VCol>
  </VRow>
  <ImageUpload/>
</template>

<style scoped>
.pic-button--text__categories {
  margin-bottom: var(--m);
}

:deep(.pic-filter-categories) {
  margin: var(--s) 0;
}
</style>

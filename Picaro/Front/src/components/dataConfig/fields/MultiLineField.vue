<script lang="ts" setup>
import {computed, ref, watch} from "vue";
import {FieldParams} from "@types";

const emit = defineEmits(["updateData", "saveEdit", "endEdit"])
const props = defineProps<{
  fieldParams: FieldParams
  fieldContent: string[]
  index: number
  isEdit: boolean

}>()
const state = ref<{ textList: string[] }>({textList: [""]})
state.value.textList = props.fieldContent ?? [""]
const formattedData = computed(() => {
  return [
    props.fieldParams.id,
    state.value.textList.filter(item => item !== ""),
  ]
})

watch(() => state.value.textList, () => {
  emit("updateData", formattedData.value);
})

function handleLines(text: string, index: number) {
  state.value.textList[index] = text
  if (text && state.value.textList.at(-1) !== "") {
    state.value.textList.push("")
  }
}

function deleteLine(index: number) {
  if (state.value.textList.length > 1) {
    state.value.textList.splice(index, 1)
  }
}

if (props.isEdit) {
  state.value.textList = props.fieldContent ?? [""];
}

</script>
<template>
  <div class="pic-multiline">
    <VTextField
      v-for="(text, index) in state.textList"
      :append-icon="state.textList.length > 1 ? 'mdi-delete' : ''"
      :hide-details="true"
      :label="fieldParams.label"
      :model-value="text"
      :name="fieldParams.name"
      data-testid="text-field"
      variant="outlined"
      @update:model-value="handleLines($event, index)"
      @click:append="deleteLine(index)"
    />
  </div>
</template>
<style lang="postcss">
.pic-multiline {
  border-left: 2px solid var(--secondaryLight);
  padding-left: var(--l);

  .v-input {
    margin-bottom: var(--l);

  }
}
</style>

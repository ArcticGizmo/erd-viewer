<template>
  <div class="by-schema">
    <multiselect v-model="selected" :options="schemas" mode="multiple">
      <template v-slot:multiplelabel="{ values }">
        <span v-for="value in values" :key="value.value" class="item">
          <span>{{ value.label }}</span>
          <span class="x-close" @click="onRemoveSelected(value.value)">x</span>
        </span>
      </template>
    </multiselect>
    <ErdViewer :nodeData="filteredNodeData" :linkData="linkData" />
  </div>
</template>

<script>
import Multiselect from '@vueform/multiselect';
import ErdViewer from './ErdViewer.vue';

function uniq(arr) {
  return [...new Set(arr)];
}

function filterNodeData(data, selected) {
  return data.filter(d => selected.includes(d.data.schema));
}

export default {
  components: {
    Multiselect,
    ErdViewer,
  },
  props: {
    nodeData: { type: Array, default: () => [] },
    linkData: { type: Array, default: () => [] },
  },
  data() {
    return {
      selected: ['Planning'],
    };
  },
  computed: {
    schemas() {
      return uniq(this.nodeData.map(r => r.data.schema)).sort((a, b) => a.localeCompare(b));
    },
    filteredNodeData() {
      return filterNodeData(this.nodeData, this.selected);
    },
  },
  methods: {
    onRemoveSelected(value) {
      this.selected = this.selected.filter(s => s !== value);
    },
  },
};
</script>

<style src="@vueform/multiselect/themes/default.css"></style>

<style scoped>
.multiselect {
  justify-content: flex-start;
}

.multiselect .item {
  background-color: rgb(175, 238, 181);
  margin: 0.25rem;
  padding: 0.25rem;
  border: 1px solid gray;
  border-radius: 5px;
}

.multiselect .item .x-close {
  cursor: pointer;
  margin-left: 0.5rem;
}

.multiselect .item .x-close:hover {
  cursor: pointer;
  margin-left: 0.5rem;
  opacity: 0.75;
}

.multiselect .multiselect-clear {
  position: absolute;
  right: 1.5rem;
}

.multiselect .multiselect-caret {
  position: absolute;
  right: 0;
}
</style>

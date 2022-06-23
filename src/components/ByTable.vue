<template>
  <div class="by-table">
    <multiselect
      v-model="selected"
      :options="tables"
      mode="multiple"
      :searchable="true"
      :clearOnSelect="false"
      :closeOnSelect="false"
    />
    <button @click="onUpdateFromSelection">Update from selection</button>
    <ErdViewer :nodeData="localNodeData" :linkData="linkData" />
  </div>
</template>

<script>
import Multiselect from '@vueform/multiselect';
import ErdViewer from './ErdViewer.vue';

function uniq(arr) {
  return [...new Set(arr)];
}

function filterNodeData(data, selected) {
  return data.filter(d => selected.includes(d.data.name));
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
      localNodeData: [],
    };
  },
  watch: {
    nodeData: {
      immediate: true,
      handler(data) {
        this.localNodeData = data || [];
      },
    },
  },
  computed: {
    tables() {
      return uniq(this.nodeData.map(r => r.data.name)).sort((a, b) => a.localeCompare(b));
    },
  },
  methods: {
    onRemoveSelected(value) {
      this.selected = this.selected.filter(s => s !== value);
    },
    onUpdateFromSelection() {
      this.localNodeData = filterNodeData(this.nodeData, this.selected);
    },
  },
};
</script>

<style src="@vueform/multiselect/themes/default.css"></style>

<style></style>

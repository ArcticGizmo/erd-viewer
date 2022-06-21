<template>
  <div class="app">
    <multiselect v-model="selected" :options="schemas" mode="multiple">
      <template v-slot:multiplelabel="{ values }">
        <span v-for="value in values" :key="value.value" class="item">
          <span>{{ value.label }}</span>
          <span class="x-close" @click="onRemoveSelected(value.value)">x</span>
        </span>
      </template>
    </multiselect>
    <!-- Add a multi select that determines what node data to display -->
    <ErdViewer :nodeData="filteredNodeData" :linkData="linkData" />
  </div>
</template>

<script>
import Multiselect from '@vueform/multiselect';
import ErdViewer from './components/ErdViewer.vue';

import COLUMN_NAMES from './files/column_names.csv';
import FOREIGN_KEYS from './files/foreign_keys.csv';

function createData(columns, foreignKeys) {
  const tables = columnsToTables(columns);

  const linkData = foreignKeysToLinks(foreignKeys);

  // need to pass in links so that items can be colored/styled correctly
  const nodeData = tablesToNodeData(tables);

  return { linkData, nodeData };
}

function columnsToTables(columns) {
  // group columns into tables
  const lookup = {};
  columns.forEach(row => {
    const key = `${row.TABLE_SCHEMA}|${row.TABLE_NAME}`;
    lookup[key] = lookup[key] || [];
    lookup[key].push({
      name: row.COLUMN_NAME,
      isKey: row.COLUMN_NAME === 'Id',
      type: row.DATA_TYPE,
      nullable: row.IS_NULLABLE,
    });
  });

  return Object.entries(lookup).map(([key, columns]) => {
    const [schema, tableName] = key.split('|');
    return {
      key,
      name: tableName,
      schema,
      columns,
    };
  });
}

function tablesToNodeData(tables) {
  return tables.map(t => {
    const items = t.columns.map(c => {
      const figure = c.isKey ? 'Decision' : 'Hexagon';
      const color = c.isKey ? 'orange' : 'blue';
      return {
        name: c.name,
        isKey: c.isKey,
        figure,
        color,
        data: c,
      };
    });

    return {
      key: t.key,
      data: {
        name: t.name,
        schema: t.schema,
      },
      items,
    };
  });
}

function foreignKeysToLinks(rows) {
  return rows.map(r => {
    const fromName = `${r.schema_name}|${r.referenced_table}`;
    const toName = `${r.schema_name}|${r.table}`;

    return {
      from: fromName,
      to: toName,
      text: 'Apple',
      toText: 'b',
      fromText: 'a',
      data: r,
    };
  });
}

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
  data() {
    const { linkData, nodeData } = createData(COLUMN_NAMES, FOREIGN_KEYS);
    return {
      linkData,
      nodeData,
      selected: [],
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

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

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

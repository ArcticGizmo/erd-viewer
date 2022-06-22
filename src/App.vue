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
    <ErdViewer :nodeData="filteredNodeData" :linkData="linkData" />
  </div>
</template>

<script>
import Multiselect from '@vueform/multiselect';
import ErdViewer from './components/ErdViewer.vue';

import COLUMN_NAMES from './files/column_names.csv';
import FOREIGN_KEYS from './files/foreign_keys.csv';

const COLORS = {
  primary: 'Green',
  foreign: 'Orange',
  value: 'Blue',
};

function createData(columns, foreignKeys) {
  const tables = columnsToTables(columns);

  const linkData = foreignKeysToLinks(foreignKeys);

  // need to pass in links so that items can be colored/styled correctly
  const nodeData = tablesToNodeData(tables, linkData);

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
      isKey: row.COLUMN_NAME === 'Id' || row.COLUMN_NAME === 'GUID',
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

function tablesToNodeData(tables, linkData) {
  return tables.map(t => {
    const items = t.columns.map(c => {
      const { color, link } = getNodeStyle(t, c, linkData);
      return {
        name: c.name,
        isKey: c.isKey,
        color,
        column: c,
        table: t,
        link,
      };
    });

    return {
      type: 'node',
      key: t.key,
      data: {
        name: t.name,
        schema: t.schema,
      },
      items,
    };
  });
}

function getNodeStyle(table, column, linkData) {
  let color = COLORS.value;
  let link;
  if (column.isKey) {
    color = COLORS.primary;
  } else {
    link = linkData.find(c => c.to === table.key && c.data.column === column.name);
    if (link) {
      color = COLORS.foreign;
    }
  }

  return {
    color,
    link,
  };
}

function foreignKeysToLinks(rows) {
  return rows.map(r => {
    const fromName = `${r.schema_name}|${r.referenced_table}`;
    const toName = `${r.schema_name}|${r.table}`;

    const text = r.referenced_column.toLowerCase() === 'id' ? '' : r.column;

    return {
      type: 'link',
      from: fromName,
      to: toName,
      text,
      toText: r.column,
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

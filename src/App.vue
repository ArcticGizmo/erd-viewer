<template>
  <div class="app">
    <div class="select">
      <button @click="onSetMode('schema')">By Schema</button>
      <button @click="onSetMode('table')">By Table</button>
    </div>
    <BySchema v-if="mode === 'schema'" :nodeData="nodeData" :linkData="linkData" />
    <ByTable v-if="mode === 'table'" :nodeData="nodeData" :linkData="linkData" />
  </div>
</template>

<script>
import BySchema from './components/BySchema.vue';
import ByTable from './components/ByTable.vue';

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

export default {
  components: {
    BySchema,
    ByTable,
  },
  data() {
    const { linkData, nodeData } = createData(COLUMN_NAMES, FOREIGN_KEYS);
    return {
      linkData,
      nodeData,
      mode: 'schema',
    };
  },
  methods: {
    onSetMode(mode) {
      this.mode = mode;
    },
  },
};
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

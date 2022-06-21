<template>
  <ErdViewer :nodeData="nodeData" :linkData="linkData" />
</template>

<script>
import ErdViewer from './components/ErdViewer.vue';

import COLUMN_NAMES from './files/column_names.csv';
import FOREIGN_KEYS from './files/foreign_keys.csv';

function createData(columns, foreignKeys) {
  const tables = columnsToTables(columns);

  const linkData = foreignKeysToLinks(foreignKeys);

  // need to pass in links so that items can be colored/styled correctly
  const nodeData = tablesToNodeData(tables);

  console.dir(linkData[0])
  console.dir(nodeData[0])

  return { linkData, nodeData };
}

function columnsToTables(columns) {
  // group columns into tables
  const lookup = {};
  columns.forEach(row => {
    const key = `${row.TABLE_NAME}|${row.TABLE_SCHEMA}`;
    lookup[key] = lookup[key] || [];
    lookup[key].push({
      name: row.COLUMN_NAME,
      isKey: row.COLUMN_NAME === 'Id',
      type: row.DATA_TYPE,
      nullable: row.IS_NULLABLE,
    });
  });

  return Object.entries(lookup).map(([key, columns]) => {
    const [tableName, schema] = key.split('|');
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
    };
  });
}

export default {
  components: {
    ErdViewer,
  },
  data() {
    const { linkData, nodeData } = createData(COLUMN_NAMES, FOREIGN_KEYS);
    return {
      linkData,
      nodeData,
    };
  },
};
</script>

<script></script>

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

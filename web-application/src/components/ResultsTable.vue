<script setup>
import { ref, watch } from "vue";

const props = defineProps(["resultsData"]);

watch(
  () => props.resultsData,
  (newResults) => {
    console.log("Saw new incoming data");
    console.log(newResults);
    getTableNames(newResults["ResultSet"]["Rows"][0]["Data"]);
    getTableData(newResults["ResultSet"]["Rows"].slice(1));
  }
);

const columnNames = ref([]);
const getTableNames = (tableColumns) => {
  console.log(tableColumns);
  columnNames.value = [];
  tableColumns.forEach((column) => {
    columnNames.value.push(column["VarCharValue"]);
  });
};
const tableRows = ref([]);
const getTableData = (tableData) => {
  console.log(tableData);
  tableRows.value = [];
  tableData.forEach((row) => {
    let rowEntries = [];
    row["Data"].forEach((rowEntry) => {
      rowEntries.push(rowEntry["VarCharValue"]);
    });
    tableRows.value.push(rowEntries);
  });
};
</script>

<template>
  <div class="result-table" v-show="columnNames.length">
    <h2>Query Results</h2>
    <table>
      <thead>
        <tr>
          <th v-for="(colName, index) in columnNames" :key="index">
            {{ colName }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in tableRows" :key="row">
          <td v-for="rowEntry in row" :key="rowEntry">{{ rowEntry }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style>
h2 {
  margin: 0 0 4px 0;
  font-size: 24px;
  text-align: center;
}

.result-table {
  padding: 20px;
  background: white;
  margin: 0 12px;
  border-radius: 24px;
}
</style>

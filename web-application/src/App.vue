<script setup>
import AthenaForm from "./components/PreparedStatementForm.vue";
import SQLStatementForm from "./components/SQLStatementForm.vue";

import { ref } from "vue";
import ResultsTable from "./components/ResultsTable.vue";

const resultsData = ref();
const handleNewResults = (data) => {
  console.log("Received data:");
  console.log(data);
  resultsData.value = data;
};
</script>

<template>
  <header></header>

  <main>
    <AthenaForm @show-results="handleNewResults" />
    <SQLStatementForm @show-results="handleNewResults" />
  </main>
  <ResultsTable :results-data="resultsData" />
</template>

<style>
* {
  box-sizing: border-box;
  font-size: 20px;
}

body {
  background: #f2f3f3;
  margin: 0;
}

main {
  display: flex;
  align-items: stretch;
}

input {
  border: 1px solid #555555;
  margin: 2px 0;
  width: 20%;
}

table,
td {
  border: 1px solid #555;
}

thead,
tfoot {
  background-color: #f3f8f0;
  color: gray;
}

button {
  padding: 4px 20px;
  background-color: #e6781f;
  border: none;
  color: white;
}

button:hover {
  background-color: #eb5f07;
}

button:disabled {
  background-color: #eb5f0799;
}
</style>

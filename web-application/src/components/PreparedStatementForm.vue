<script setup>
import axios from "axios";
import { ref, watch, onMounted } from "vue";

const API_URL = import.meta.env.AWS_API_ENDPOINT;
const APP_AUTH = import.meta.env.API_AUTH_CODE;

// Functionality for retrieving a list of workgroups and prepared statements
const workgroups = ref([]);
const listPreparedStatements = () => {
  workgroups.value = [];

  const config = {
    headers: {
      Authorization: APP_AUTH,
      "Content-Type": "application/json",
    },
  };

  console.log("Retrieving a list of workgroups and prepared statements");
  axios
    .get(API_URL + "/prepared-statements", config)
    .then((response) => {
      console.log(response);
      workgroups.value = response.data;
    })
    .catch((error) => {
      console.log(error);
      errorMessage.value = error;
    });
};
onMounted(() => listPreparedStatements());

// Functionality for workgroup and statement selection
const statements = ref([]);
const workgroupPlaceholder = "Select Workgroup";
const statementPlaceholder = "Select Statement";
const selectedWorkgroup = ref(workgroupPlaceholder);
const selectedStatement = ref(statementPlaceholder);
const selectedStatementName = ref("");
const selectedStatementDescription = ref("");
const selectedQueryString = ref("");
const queryParameters = ref([]);
const inputParameters = ref([]);
const getQueryParameters = (queryString) => {
  /* Apply RegEx on the query string to pull out expected parameters
   *
   *  Example: "WHERE (((product_id = ?) AND (star_rating = ?)) AND (helpful_votes > ?))
   *            ORDER BY helpful_votes DESC LIMIT 10"
   *
   *  Retrieves: ['product_id =', 'star_rating =', 'helpful_votes >']
   * */
  queryString = queryString.replaceAll("\n", " ");
  queryString = queryString.replaceAll("(", "");
  queryString = queryString.replaceAll(")", "");
  const split = queryString.split(" ");
  const parameters = [];

  for (const [index, token] of split.entries()) {
    // console.log(token)
    if (token === "?") {
      inputParameters.value.push();
      parameters.push({
        param: `${split[index - 2]} ${split[index - 1]}`,
        inputValue: "",
      });
    }
  }
  return parameters;
};
watch(selectedWorkgroup, (newWorkgroup) => {
  statements.value = workgroups.value[newWorkgroup];
});
watch(selectedStatement, () => {
  selectedQueryString.value = selectedStatement.value["query"];
  selectedStatementName.value = selectedStatement.value["name"];
  selectedStatementDescription.value = selectedStatement.value["description"];
  queryParameters.value = getQueryParameters(selectedQueryString.value);
});

// Functionality related to querying data
const emit = defineEmits(["showResults"]);
const results = ref("");
const errorMessage = ref("");
const submitBtn = ref(null);
const getData = () => {
  submitBtn.value.innerText = "Fetching...";
  submitBtn.value.disabled = true;
  errorMessage.value = "";

  let parameters = [];

  queryParameters.value.forEach((item) => {
    console.log(item);
    parameters.push(item.inputValue.toString());
  });

  const data = JSON.stringify({
    type: "prepared",
    statement: selectedStatementName.value,
    parameters,
    workgroup: selectedWorkgroup.value,
  });
  console.log(data);

  const config = {
    headers: {
      Authorization: APP_AUTH,
      "Content-Type": "application/json",
    },
  };

  axios
    .post(API_URL, data, config)
    .then((response) => {
      console.log(response.data);
      if (!response.data["ResultSet"]) {
        errorMessage.value = "Query Failed";
      } else {
        // Query ran successfully
        results.value = response.data;
        console.log("PreparedStatementForm is emitting event");
        emit("showResults", results.value);
      }
      submitBtn.value.disabled = false;
      submitBtn.value.innerText = "Run Query";
    })
    .catch((error) => {
      submitBtn.value.disabled = false;
      submitBtn.value.innerText = "Run Query";
      console.log(error);
      errorMessage.value = error.response.data ? error.response.data : error;
    });
};
</script>

<template>
  <section>
    <h2>Parameterized Prepared Statements</h2>
    <hr />
    <form>
      <div>
        <label for="">Workgroup Name: </label>
        <select v-model="selectedWorkgroup">
          <option disabled>{{ workgroupPlaceholder }}</option>
          <option
            v-for="workgroup in Object.keys(workgroups)"
            :key="workgroup"
            :value="workgroup"
          >
            {{ workgroup }}
          </option>
        </select>
      </div>

      <div>
        <label for="">Prepared Statement: </label>
        <select
          v-model="selectedStatement"
          :disabled="selectedWorkgroup === 'Select Workgroup'"
        >
          <option disabled>{{ statementPlaceholder }}</option>
          <option
            v-for="(statement, index) in statements"
            :key="index"
            :value="statement"
            :selected="index === 0"
          >
            {{ statement["name"] }}
          </option>
        </select>
      </div>

      <div v-show="selectedStatementDescription">
        <h3>Description:</h3>
        {{ selectedStatementDescription }}
      </div>

      <div v-show="selectedQueryString">
        <h3>Query string of prepared statement:</h3>
        <p class="snippet">{{ selectedQueryString }}</p>
      </div>
      
      <form class="param-form">
        <div v-for="parameter in queryParameters" :key="parameter">
          <label class="param-label">{{ parameter.param }}</label>
          <input class="param-input" v-model="parameter.inputValue" />
        </div>
      </form>
    </form>

    <button
      class="btn-query"
      v-show="selectedQueryString"
      ref="submitBtn"
      @click="getData"
    >
      Run Query
    </button>

    <p v-show="errorMessage">{{ errorMessage }}</p>
  </section>
</template>

<style scoped>
section {
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: white;
  margin: 24px 12px;
  border-radius: 24px;
  width: 50%;
}

h2 {
  margin: 0;
  font-size: 24px;
  text-align: center;
}

select {
  font-size: 18px;
  margin: 4px 0;
  padding: 4px 10px;
  border: 1px solid #555555;
  background: white;
}

.snippet {
  padding: 8px;
  background: lightgrey;
}

.param-form {
  margin: 10px 0;
}

.param-label {
  margin-right: 12px;
}

.param-input {
  min-width: fit-content;
}

.btn-query {
  margin-left: auto;
}
</style>

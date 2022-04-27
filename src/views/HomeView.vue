<template>
  <div class="home">
    <input v-model="searchInput" placeholder="Search" />

    <ApolloQuery
      :query="
        searchInput != ''
          ? (gql) => gql`
              query Search($id: Int!) {
                todo_by_pk(id: $id) {
                  text
                  id
                }
              }
            `
          : (gql) => gql`
              query MyQuery {
                todo {
                  text
                  id
                }
              }
            `
      "
      :variables="search"
    >
      <template v-slot="{ result: { loading, error, data } }">
        <!-- Loading -->
        <div v-if="loading" class="loading apollo">Loading...</div>

        <!-- Error -->
        <div v-else-if="error" class="error apollo">An error occurred</div>

        <!-- Result -->
        <div v-else-if="data" class="result apollo">
          <div v-if="data.todo">
            <div v-for="Todo in data.todo" :key="Todo.id">
              <ul>
                <li>
                  {{ Todo.text }}
                </li>
              </ul>
            </div>
          </div>
          <div v-else-if="data.todo_by_pk">{{ data.todo_by_pk.text }}</div>
          <div v-else-if="data.todo_by_pk == null && !data.todo">
            {{ "Belum ada todo" }}
          </div>
        </div>

        <!-- No result -->
        <div v-else class="no-result apollo">No result :(</div>
      </template>
    </ApolloQuery>
  </div>
</template>

<script>
export default {
  name: "HomeView",
  data() {
    return {
      todo: "",
      searchInput: "",
    };
  },
  computed: {
    search() {
      if (this.searchInput !== "") {
        return { id: this.searchInput };
      }
      return {};
    },
  },
};
</script>

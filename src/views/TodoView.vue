<template>
  <!-- eslint-disable prettier/prettier -->
  <div class="about">
    <ApolloMutation
      :mutation="
        (gql) => gql`
          mutation MyMutation($text: String!) {
            insert_todo_one(object: { text: $text }) {
              text
              id
            }
          }
        `
      "
      :variables="{
        text: inputTodo,
      }"
    >
      <template v-slot="{ mutate, loading, error }">
        <div v-if="isEnabled">
          <input v-model="inputTodo" placeholder="Input" />
          <button :disabled="loading" @click="mutate()">Add ToDo</button>
          <button @click="edit()">Edit ToDo</button>
          <p v-if="error">An error occurred: {{ error }}</p>
        </div>
      </template>
    </ApolloMutation>
    <hr />
    <div v-if="isEdit">
      <h1>EDIT</h1>
      <ApolloQuery
        :query="
          (gql) => gql`
            query MyQuery {
              todo {
                text
                id
              }
            }
          `
        "
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
                    {{ Todo.id }}
                    {{ Todo.text }}
                    <ApolloMutation
                      :mutation="
                        (gql) => gql`
                          mutation MyMutation2($id: Int!) {
                            delete_todo_by_pk(id: $id) {
                              text
                              id
                            }
                          }
                        `
                      "
                      :variables="{
                        id: Todo.id,
                      }"
                    >
                      <template v-slot="{ mutate, loading, error }">
                        <input type="button" :disabled="loading" value="Hapus" @click="mutate()" />
                        <p v-if="error">An error occurred: {{ error }}</p>
                      </template>
                    </ApolloMutation>
                    <ApolloMutation
                      :mutation="
                        (gql) => gql`
                          mutation MyMutation3($id: Int!, $text: String!) {
                            update_todo_by_pk(pk_columns: { id: $id }, _set: { text: $text }) {
                              text
                              id
                            }
                          }
                        `
                      "
                      :variables="{
                        id: Todo.id,
                        text: editTodo,
                      }"
                    >
                      <template v-slot="{ mutate, loading, error }">
                        <div v-if="indexSelected === Todo.id" id="list-edit">
                          <input type="text" v-model="editTodo" /><br />
                          <input :disabled="loading" @click="mutate()" type="button" value="Update" />
                        </div>
                        <!--eslint-disable-next-line prettier/prettier-->
                        <input v-if="isEdited" @click="EditTodo(Todo.text, Todo.id)" type="button" value="Edit" />

                        <p v-if="error">An error occurred: {{ error }}</p>
                      </template>
                    </ApolloMutation>
                  </li>
                </ul>
              </div>
            </div>
          </div>

          <!-- No result -->
          <div v-else class="no-result apollo">No result :(</div>
        </template>
      </ApolloQuery>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      inputTodo: "",
      isEnabled: true,
      isEdit: false,
      isEdited: true,
      editTodo: "",
      indexSelected: null,
    };
  },
  methods: {
    edit() {
      this.isEnabled = false;
      this.isEdit = true;
    },
    EditTodo(text, id) {
      this.editTodo = text;
      this.indexSelected = id;
      this.isEdited = false;
    },
  },
};
</script>

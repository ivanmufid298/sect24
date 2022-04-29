<template>
  <!-- eslint-disable prettier/prettier -->
  <div class="about">
    <ApolloMutation
      :mutation="
        (gql) => gql`
          mutation MyMutation($name: String!) {
            insert_todo_one(object: { name: $name }) {
              name
              id
            }
          }
        `
      "
      :variables="{
        name: inputTodo,
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
                id 
                name
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
                    {{ Todo.name }}
                    <ApolloMutation
                      :mutation="
                        (gql) => gql`
                          mutation MyMutation2($id: Int!) {
                            delete_todo_by_pk(id: $id) {
                              id
                              name
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
                          mutation MyMutation3($id: Int!, $name: String!) {
                            update_todo_by_pk(pk_columns: { id: $id }, _set: { name: $name }) {
                              id
                              name
                            }
                          }
                        `
                      "
                      :variables="{
                        id: Todo.id,
                        name: editTodo,
                      }"
                    >
                      <template v-slot="{ mutate, loading, error }">
                        <div v-if="indexSelected === Todo.id" id="list-edit">
                          <input type="name" v-model="editTodo" /><br />
                          <input :disabled="loading" @click="mutate()" type="button" value="Update" />
                        </div>
                        <!--eslint-disable-next-line prettier/prettier-->
                        <input v-if="isEdited" @click="EditTodo(Todo.name, Todo.id)" type="button" value="Edit" />

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
    EditTodo(name, id) {
      this.editTodo = name;
      this.indexSelected = id;
      this.isEdited = false;
    },
  },
};
</script>

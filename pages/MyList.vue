<template>
  <div>
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-text-field
        v-model="todo.name"
        :rules="nameRules"
        label="Name"
        required
      ></v-text-field>

      <v-text-field
        v-model="todo.description"
        label="Description"
        required
      ></v-text-field>

      <v-btn :disabled="!valid" color="success" class="mr-4" @click="create()">
        Submit
      </v-btn>

      <v-btn color="error" class="mr-4" @click="reset"> Reset Form </v-btn>

      <v-btn color="warning" @click="resetValidation"> Reset Validation </v-btn>
    </v-form>

    <v-data-table
      :headers="headers"
      :items="todos"
      :items-per-page="10"
      class="elevation-1 mt-2"
    >
      <template #item.status="{ item }">
        <v-chip :color="item.status == 'pending' ? 'red' : 'green'">{{
          item.status
        }}</v-chip>
        <!-- <v-chip color="red" v-if="item.status == 'pending'">{{
          item.status
        }}</v-chip>
        <v-chip color="green" v-else>{{ item.status }}</v-chip> -->
      </template>
      <template #item.created="{ item }">
        {{ setDate(item.created_at) }}
      </template>
      <template #item.actions="{ item }">
        <v-btn color="dark" @click="setAsDone(item.id)">SET AS DONE</v-btn>

        <!-- edit dialog -->
        <v-dialog v-model="dialog[item.id]" width="500">
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              v-bind="attrs"
              v-on="on"
              @click="findById(item.id)"
            >
              Edit
            </v-btn>
          </template>

          <v-card>
            <v-card-title class="text-h5 lighten-2">
              {{ item.name }}
            </v-card-title>

            <v-card-text>
              <v-text-field
                v-model="updateTodo.name"
                label="Name"
                required
              ></v-text-field>

              <v-text-field
                v-model="updateTodo.description"
                label="Description"
                required
              ></v-text-field>
            </v-card-text>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" @click="update(item.id)">SAVE</v-btn>
              <v-btn color="error" text @click="$set(dialog, item.id, false)">
                Close
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- end edit dialog -->
        <v-btn color="error" @click="remove(item.id)">DELETE</v-btn>
      </template>
    </v-data-table>
  </div>
</template>

<script>
export default {
  data: () => ({
    todos: [],
    todo: {
      name: "",
      description: "",
    },
    updateTodo: {
      name: "",
      description: "",
    },
    dialog: [],
    valid: true,
    nameRules: [
      (v) => !!v || "Name is required",
      (v) => (v && v.length <= 10) || "Name must be less than 10 characters",
    ],
    email: "",
    emailRules: [
      (v) => !!v || "E-mail is required",
      (v) => /.+@.+\..+/.test(v) || "E-mail must be valid",
    ],
    headers: [
      {
        text: "Name",
        align: "start",
        sortable: false,
        value: "name",
      },
      { text: "Desc", value: "description" },
      { text: "Status", value: "status" },
      { text: "Created at", value: "created" },
      { text: "Actions", value: "actions" },
    ],
  }),
  mounted() {
    this.getTodos();
  },
  methods: {
    validate() {
      this.$refs.form.validate();
    },
    reset() {
      this.$refs.form.reset();
    },
    resetValidation() {
      this.$refs.form.resetValidation();
    },
    setDate(created_at) {
      return new Date(
        new Date(created_at) - new Date().getTimezoneOffset() * 60000
      )
        .toISOString()
        .substr(0, 10);
    },
    async getTodos() {
      try {
        const res = await this.$axios.get("/todo");
        if (res.status === 200) {
          this.todos = res.data.todos;
        }
      } catch (err) {
        console.log(err.response);
      }
    },
    async create() {
      try {
        const res = await this.$axios.post("/todo", this.todo);
        if (res.status === 201) {
          this.getTodos();
          this.reset();
        }
      } catch (err) {
        console.log(err.response);
      }
    },
    async setAsDone(id) {
      try {
        const res = await this.$axios.put(`/todo/done/${id}`);
        if (res.status === 200) {
          this.getTodos();
        }
      } catch (err) {
        console.log(err.response);
      }
    },
    async remove(id) {
      try {
        const res = await this.$axios.delete(`/todo/${id}`);
        if (res.status === 200) {
          this.getTodos();
        }
      } catch (err) {
        console.log(err.response);
      }
    },
    async findById(id) {
      try {
        const res = await this.$axios.get(`/todo/${id}`);
        if (res.status === 200) {
          this.updateTodo.name = res.data.todo.name;
          this.updateTodo.description = res.data.todo.description;
        }
      } catch (err) {
        console.log(err.response);
      }
    },
    async update(id) {
      try {
        const res = await this.$axios.put(`/todo/${id}`, this.updateTodo);
        if (res.status === 200) {
          this.getTodos();
          this.dialog[id] = false;
        }
      } catch (err) {
        console.log(err.response);
      }
    },
  },
};
</script>
<template>
  <v-container>
    <v-row class="text-center">
      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          Welcome to To-do's
        </h1>
        <v-theme-provider root :dark="isDark">
          <v-container>
            <v-row justify="center" class="ma-5">
              <v-col xs="12" sm="8">
                <v-card>
                  <v-toolbar color="blue darken-4" dark>
                    <v-toolbar-title class="headline">Todo App</v-toolbar-title>
                    
                    <v-spacer></v-spacer>

                    <v-text-field
                      v-model="search"
                      append-icon="mdi-magnify"
                      label="Search"
                      single-line
                      hide-details
                    ></v-text-field>
                    <v-divider
                          class="mx-4"
                          inset
                          vertical
                    ></v-divider>

                    <v-tooltip bottom>
                      <template v-slot:activator="{ on }">
                        <v-btn icon @click="isDark = !isDark" v-on="on">
                          <v-icon v-model="isDark">{{ !isDark ? 'mdi-weather-night' : 'mdi-weather-cloudy' }}</v-icon>
                        </v-btn>
                      </template>
                      <span>
                        {{ isDark ? 'light mode' : 'dark mode' }}
                      </span>
                    </v-tooltip>
                  </v-toolbar>


                  <v-data-table
                    v-model="selected"
                    :headers="headers"
                    :items="todos"
                    sort-by="calories"
                    class="elevation-1"
                    hide-default-footer
                    :search="search"
                    show-select
                    item-key="name"
                  >
                    <template v-slot:top>
                      <v-toolbar
                        flat
                      >

                        <v-toolbar-title class="subheading" v-if="todos.length == 0">You have 0 Tasks, add some</v-toolbar-title>
                        <v-toolbar-title class="subheading" v-else>Your Tasks</v-toolbar-title>
                        <v-divider
                          class="mx-4"
                          inset
                          vertical
                        ></v-divider>

                        <v-text-field class="mt-5" v-model="newTodo" id="newTodo" name="newTodo" label="Type and press enter to add your task" @keyup.enter="addTodo" />

                        <v-btn v-if="selected.length > 1"
                            color="red darken-1"
                            text
                            @click="deleteAll"
                          >
                          Delete all
                        </v-btn>

                        <v-dialog
                          v-model="dialog"
                          max-width="500px"
                        >
                          
                          <v-card>
                            <v-card-text>
                              <v-container>
                                <v-row>
                                  <v-col
                                    cols="12"
                                    sm="12"
                                    md="12"
                                  >
                                    <v-text-field
                                      v-model="editedItem.name"
                                      label="Todo"
                                    ></v-text-field>
                                  </v-col>
                                </v-row>
                              </v-container>
                            </v-card-text>
                
                            <v-card-actions>
                              <v-spacer></v-spacer>
                              <v-btn
                                color="red darken-1"
                                text
                                @click="close"
                              >
                                Cancel
                              </v-btn>
                              <v-btn
                                color="green darken-1"
                                text
                                @click="save"
                              >
                                Update
                              </v-btn>
                            </v-card-actions>
                          </v-card>
                        </v-dialog>
                        <v-dialog v-model="dialogDelete" max-width="500px">
                          <v-card>
                            <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
                            <v-card-actions>
                              <v-spacer></v-spacer>
                              <v-btn color="red darken-1" text @click="closeDelete">Cancel</v-btn>
                              <v-btn color="green darken-1" text @click="deleteItemConfirm">OK</v-btn>
                              <v-spacer></v-spacer>
                            </v-card-actions>
                          </v-card>
                        </v-dialog>
                      </v-toolbar>
                    </template>
                    <template class="text-right" v-slot:item.actions="{ item }">
                      <v-icon
                        small
                        class="mr-2"
                        color="warning"
                        @click="editItem(item)"
                      >
                        mdi-pencil
                      </v-icon>
                      <v-icon
                        small
                        color="red"
                        @click="deleteItem(item)"
                      >
                        mdi-delete
                      </v-icon>
                    </template>
                  </v-data-table>
                </v-card>
              </v-col>
            </v-row>
          </v-container>
        </v-theme-provider>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'Todo',
    data: () => ({
      selected: [],
      search: '',
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          text: 'Title',
          align: 'start',
          sortable: true,
          value: 'name',
        },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      editedIndex: -1,
      editedItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
      defaultItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
      isDark: true,
      show: true,
      newTodo: "",
      todos: [],
      active:''
    }),
    methods:{
      deleteAll() {
        this.todos = [];
      },
      editItem (item) {
        this.editedIndex = this.todos.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },
      deleteItem (item) {
        this.editedIndex = this.todos.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },
      deleteItemConfirm () {
        this.todos.splice(this.editedIndex, 1)
        this.closeDelete()
      },
      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },
      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },
      save () {
        if (this.editedIndex > -1) {
          Object.assign(this.todos[this.editedIndex], this.editedItem)
        } else {
          this.todos.push(this.editedItem)
        }
        console.log(this.todos);
        this.close()
      },
      addTodo() {
        const value = this.newTodo && this.newTodo.trim();
        if (!value) {
          return;
        }

        this.editedItem.name = value;
        this.todos.push(this.editedItem)
        this.newTodo = "";
        console.log(this.todos);
        this.close()
      }
  }
}  
</script>

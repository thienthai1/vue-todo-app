<template>
  <div id="app" class="max-w-2xl mx-auto p-6 font-sans bg-white text-black">
    <div class="container">
      <h1 class="text-3xl font-bold text-center mb-6">Todo List App</h1>
      <div class="text-center mb-6">
        <h2 class="text-xl">To Do ({{ todosCount }}) Done ({{ doneCount }})</h2>
      </div>
      <!-- Todo Input Form -->
      <div class="flex items-center justify-center mb-6">
        <input type="text" v-model="newTodo" placeholder="Enter a new todo" class="border rounded px-4 py-2 mr-2 w-64">
        <button class="bg-blue-500 text-white rounded px-4 py-2" @click="addTodo">Add Todo</button>
      </div>
      <!-- Todo Section -->
      <div>
        <h2 class="text-2xl mb-4">To Do</h2>
        <div v-if="todos.length === 0" class="text-gray-500">No todos yet.</div>
        <div v-else>
          <div v-for="(todo, index) in todos" :key="index" class="flex items-center mb-4">
            <template v-if="!todo.editMode">
              <span class="flex-1">{{ "- " + todo.text }}</span>
              <button class="bg-red-500 text-white rounded px-2 py-1 ml-2" @click="confirmDelete(todo, 'todo')">Delete</button>
              <button class="bg-yellow-500 text-white rounded px-2 py-1 ml-2" @click="toggleEditMode(todo)">Edit</button>
            </template>
            <template v-else>
              <input type="text" v-model="todo.text" @keyup.enter="updateTodo(index)" class="flex-1 border rounded px-2 py-1 mr-2">
              <button class="bg-green-500 text-white rounded px-2 py-1 ml-2" @click="updateTodo(index)">Save</button>
              <button class="bg-gray-500 text-white rounded px-2 py-1 ml-2" @click="cancelEdit(todo)">Cancel</button>
            </template>
            <button class="bg-green-500 text-white rounded px-2 py-1 ml-2" @click="moveToDone(index)">Done</button>
          </div>
        </div>
      </div>
      <!-- Done Section -->
      <div>
        <h2 class="text-2xl mb-4">Done</h2>
        <div v-if="doneTodos.length === 0" class="text-gray-500">No todos are done yet.</div>
        <div v-else>
          <div v-for="(todo, index) in doneTodos" :key="index" class="flex items-center mb-4">
            <span class="flex-1">{{ "- " + todo.text }}</span>
            <button class="bg-red-500 text-white rounded px-2 py-1 ml-2" @click="confirmDelete(todo, 'done')">Delete</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Confirmation Modal -->
  <confirmation-modal 
      v-if="showModal" 
      :show="showModal" 
      :onConfirm="deleteConfirmed" 
      :onCancel="cancelDelete" />
</template>

<script>
import { database } from './firebase';
import { getDatabase, ref, onValue, push, set, remove } from "firebase/database";
import ConfirmationModal from './ConfirmationModal.vue';

export default {
  components: {
    ConfirmationModal
  },
  data() {
    return {
      todos: [],
      doneTodos: [],
      newTodo: '',
      showModal: false,
      todoToDelete: null,
      deleteType: ''
    };
  },
  created() {
    this.fetchTodos();
  },
  computed: {
    todosCount() {
      return this.todos.length;
    },
    doneCount() {
      return this.doneTodos.length;
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim() !== '') {
        const db = getDatabase();
        const dbRef = ref(db, 'todos/');
        const newTodo = { text: this.newTodo, finished: false, editMode: false };
        const newPostRef = push(dbRef);
        set(newPostRef, newTodo)
        this.newTodo = '';
      }
    },
    fetchTodos() {
      const db = getDatabase();
      const dbRef = ref(db, 'todos/');
      onValue(dbRef, (snapshot) => {
        const todos = [];
        const doneTodos = [];
        snapshot.forEach(childSnapshot => {
          const todo = childSnapshot.val();
          todo.key = childSnapshot.key;
          if (todo.finished) {
            doneTodos.push(todo);
          } else {
            todos.push(todo);
          }
        });
        this.todos = todos;
        this.doneTodos = doneTodos;
      });
    },
    updateTodo(index) {
      const todo = this.todos[index];
      if (todo.text.trim() === '') {
        this.deleteTodoFromDb(todo.key);
      } else {
        todo.editMode = false;
        const db = getDatabase();
        const dbRef = ref(db, `todos/${todo.key}`);
        set(dbRef, todo)
      }
    },
    confirmDelete(todo, type) {
      console.log("testing")
      this.todoToDelete = todo;
      this.deleteType = type;
      this.showModal = true;
    },
    deleteConfirmed() {
      if (this.deleteType === 'todo') {
        this.deleteTodoFromDb(this.todoToDelete.key);
      } else if (this.deleteType === 'done') {
        this.deleteTodoFromDb(this.todoToDelete.key);
      }
      this.showModal = false;
    },
    cancelDelete() {
      this.showModal = false;
      this.todoToDelete = null;
      this.deleteType = '';
    },
    deleteTodo(index) {
      const todo = this.todos[index];
      this.deleteTodoFromDb(todo.key);
    },
    deleteTodoFromDb(key) {
      const db = getDatabase();
      const dbRef = ref(db, `todos/${key}`);
      remove(dbRef)
    },
    moveToDone(index) {
      const todo = this.todos[index];
      todo.finished = true;
      const db = getDatabase();
      const dbRef = ref(db, `todos/${todo.key}`);
      set(dbRef, todo)
    },
    deleteDone(index) {
      const todo = this.doneTodos[index];
      this.deleteTodoFromDb(todo.key);
    },
    toggleEditMode(todo) {
      todo.editMode = true;
    },
    cancelEdit(todo) {
      todo.editMode = false;
    },
  }
};
</script>


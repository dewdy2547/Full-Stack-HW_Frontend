<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";

// Define the type for a Todo item based on the database schema
type Todo = {
  id: number;
  title: string;
  description?: string;
  completed: boolean;
  created_at: string;
};

//Import backend URL
const baseURL = import.meta.env.VITE_BASE_URL;
console.log(baseURL);

// Create a reactive reference to hold the list of todos
const todos = ref<Todo[]>([]);
const todoText = ref("");
const todoDescription = ref("");
const edit = ref(false);
const currentTodo = ref<Todo | null>(null);

async function fetchTodos() {
  const res = await axios.get(`${baseURL}/todos`);
  console.log(res);
  todos.value = res.data || [];
}

async function handleSubmitTodo() {
  if (!todoText.value) return;

  if (!edit.value && currentTodo.value) {
    await axios.post(`${baseURL}/${currentTodo.value.id}`, {
      title: todoText.value,
      description: todoDescription.value
    });
    fetchTodos();
    todoText.value = "";
    todoDescription.value = "";
  }
  if (edit.value && currentTodo.value) {
    await axios.patch(`${baseURL}/todos/${currentTodo.value.id}`, {
      title: todoText.value,
      description: todoDescription.value,
    });
    fetchTodos();
    todoText.value = "";
    edit.value = false;
    currentTodo.value = null;
  }
}

async function handleDeleteTodo(id: number) {
  await axios.delete(`${baseURL}/todos/${id}`);
  fetchTodos();
}

async function toggleCompleted(todo: Todo) {
  await axios.patch(`${baseURL}/todos/${todo.id}`, {
    completed: !todo.completed,
  });
  fetchTodos();
}

function handleEditTodo(todo: Todo) {
  edit.value = true;
  currentTodo.value = todo;
  todoText.value = todo.title;
  todoDescription.value = todo.description || "";
}

function cancelEdit() {
  edit.value = false;
  currentTodo.value = null;
  todoText.value = "";
  todoDescription.value = "";
}

// Fetch the todos when the component is mounted
fetchTodos();
</script>

<template>
  <div class="main-container">
    <div class="todo-card">
      <h1>To Do List</h1>
      
      <!-- Form Input Section -->
      <div class="form-section">
        <input type="text" v-model="todoText" placeholder="Add a new todo title..." class="input-field" />
        <input type="text" v-model="todoDescription" placeholder="Add a description..." class="input-field" />
        <div class="btn-group">
          <button @click="handleSubmitTodo" class="btn-submit">{{ edit ? "Update" : "Add" }}</button>
          <button v-if="edit" @click="cancelEdit" class="btn-cancel">Cancel</button>
        </div>
      </div>

      <!-- Todo List Wrapper -->
      <div class="todo-wrapper">
        <div v-for="(todo, idx) in todos" :key="todo.id" class="todo-item">
          <div class="todo-left">
            <input 
              type="checkbox" 
              :checked="todo.completed" 
              @change="toggleCompleted(todo)" 
              class="checkbox"
            />
            <span class="index">{{ idx + 1 }}.</span>
            <div class="todo-content">
              <span :class="{ 'completed-text': todo.completed }" class="title">
                📰 {{ todo.title }}
              </span>
              <p v-if="todo.description" class="description">
                💬 {{ todo.description }}
              </p>
              <div class="time-info">
                <span>📅 {{ new Date(todo.created_at).toLocaleDateString() }}</span>
                <span>⏰ {{ new Date(todo.created_at).toLocaleTimeString() }}</span>
              </div>
            </div>
          </div>

          <div class="todo-actions">
            <span @click="handleEditTodo(todo)" class="trash" title="Edit">🖊️</span>
            <span @click="handleDeleteTodo(todo.id)" class="trash" title="Delete">🗑️</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Prompt:ital,wght@0,400;0,700;1,400&display=swap");

* {
  font-family: "Prompt", sans-serif;
  box-sizing: border-box;
}

.main-container {
  min-height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 3em 1em;
  margin: 0;
}

.todo-card {
  background: #ffffff;
  padding: 2.5em;
  border-radius: 1rem;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  width: 100%;
  max-width: 600px;
}

h1 {
  text-align: center;
  color: #333;
  margin-top: 0;
  margin-bottom: 1.5em;
}

.form-section {
  display: flex;
  flex-direction: column;
  gap: 0.8em;
  margin-bottom: 2em;
}

.input-field {
  padding: 0.8em 1em;
  border: 1px solid #ddd;
  border-radius: 0.5em;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s;
}

.input-field:focus {
  border-color: #667eea;
}

.btn-group {
  display: flex;
  gap: 0.5em;
}

button {
  padding: 0.7em 1.5em;
  border: none;
  border-radius: 0.5em;
  font-weight: bold;
  cursor: pointer;
  transition: opacity 0.2s;
}

button:hover {
  opacity: 0.9;
}

.btn-submit {
  background-color: #667eea;
  color: white;
  flex: 1;
}

.btn-cancel {
  background-color: #e53e3e;
  color: white;
}

.todo-wrapper {
  display: flex;
  flex-direction: column;
  gap: 1em;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  border: 1px solid #e2e8f0;
  padding: 1em;
  border-radius: 0.8em;
  background-color: #f8fafc;
  transition: background-color 0.2s;
}

.todo-item:hover {
  background-color: #f1f5f9;
}

.todo-left {
  display: flex;
  gap: 0.8em;
  align-items: flex-start;
  flex: 1;
}

.checkbox {
  margin-top: 0.3em;
  cursor: pointer;
  width: 1.2rem;
  height: 1.2rem;
}

.index {
  font-weight: bold;
  color: #64748b;
}

.todo-content {
  display: flex;
  flex-direction: column;
  gap: 0.3em;
  flex: 1;
}

.title {
  font-weight: bold;
  color: #1e293b;
  font-size: 1.1rem;
}

.completed-text {
  text-decoration: line-through;
  color: #94a3b8;
}

.description {
  margin: 0;
  color: #475569;
  font-size: 0.95rem;
  word-break: break-word;
}

.time-info {
  display: flex;
  gap: 1em;
  font-size: 0.8rem;
  color: #94a3b8;
  margin-top: 0.2em;
}

.todo-actions {
  display: flex;
  gap: 0.8em;
  align-items: center;
}

.trash {
  cursor: pointer;
  font-size: 1.2rem;
  transition: transform 0.1s;
}

.trash:hover {
  transform: scale(1.2);
}
</style>

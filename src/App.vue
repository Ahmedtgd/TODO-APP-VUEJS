<script setup>

//import TodoApp from '/src/components/TodoApp.vue'
import { ref, computed, watchEffect } from 'vue'

const STORAGE_KEY = 'vue-todomvc'

const filters = {
  all: (todos) => todos,
  active: (todos) => todos.filter((todo) => !todo.completed),
  completed: (todos) => todos.filter((todo) => todo.completed)
}

// state
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
const visibility = ref('all')
const editedTodo = ref()

// derived state
const filteredTodos = computed(() => filters[visibility.value](todos.value))
const remaining = computed(() => filters.active(todos.value).length)

// handle routing
window.addEventListener('hashchange', onHashChange)
onHashChange()

// persist state
watchEffect(() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
})

function toggleAll(e) {
  todos.value.forEach((todo) => (todo.completed = e.target.checked))
}

function addTodo(e) {
  const value = e.target.value.trim()
  if (value) {
    todos.value.push({
      id: Date.now(),
      title: value,
      completed: false
    })
    e.target.value = ''
  }
}

function removeTodo(todo) {
  todos.value.splice(todos.value.indexOf(todo), 1)
}

let beforeEditCache = ''
function editTodo(todo) {
  beforeEditCache = todo.title
  editedTodo.value = todo
}

function cancelEdit(todo) {
  editedTodo.value = null
  todo.title = beforeEditCache
}

function doneEdit(todo) {
  if (editedTodo.value) {
    editedTodo.value = null
    todo.title = todo.title.trim()
    if (!todo.title) removeTodo(todo)
  }
}

function removeCompleted() {
  todos.value = filters.active(todos.value)
}

function onHashChange() {
  const route = window.location.hash.replace(/#\/?/, '')
  if (filters[route]) {
    visibility.value = route
  } else {
    window.location.hash = ''
    visibility.value = 'all'
  }
}
</script>

<template>
      <todo-app/>
      <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        autofocus
        placeholder="What needs to be done?"
        @keyup.enter="addTodo"
      >
    </header>
    <section class="main" v-show="todos.length">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        :checked="remaining === 0"
        @change="toggleAll"
      >
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          class="todo"
          :key="todo.id"
          :class="{ completed: todo.completed, editing: todo === editedTodo }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input
            v-if="todo === editedTodo"
            class="edit"
            type="text"
            v-model="todo.title"
            @vue:mounted="({ el }) => el.focus()"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.escape="cancelEdit(todo)"
          >
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length">
      <span class="todo-count">
        <strong>{{ remaining }}</strong>
        <span>{{ remaining === 1 ? ' item' : ' items' }} left</span>
      </span>
      <ul class="filters">
        <li>
          <a href="#/all" :class="{ selected: visibility === 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: visibility === 'active' }">Active</a>
        </li>
        <li>
          <a href="#/completed" :class="{ selected: visibility === 'completed' }">Completed</a>
        </li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
        Clear completed
      </button>
    </footer>
  </section>
</template>

<style >

/* Reset styles to normalize browser defaults */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: sans-serif;
  color: #222;
}

html, body {
  height: 100%;
  background-color: #f2f2f2;
}

/* Header section */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 40px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.header h1 {
  font-size: 32px;
  font-weight: bold;
  color: #333;
}

.new-todo {
  width: 100%;
  padding: 10px;
  border: none;
  outline: none;
  background-color: transparent;
  font-size: 18px;
}

/* Main section */
.main {
  padding: 20px 40px;
}

.toggle-all {
  position: absolute;
  top: 20px;
  left: 40px;
  opacity: 0;
}

.toggle-all:checked + label {
  text-decoration: line-through;
}

.toggle-all + label {
  cursor: pointer;
  padding: 10px;
  border: 1px solid #ccc;
  color: #999;
}

.todo-list {
  margin-top: 20px;
  list-style: none;
}

.todo {
  padding: 10px;
  border: 1px solid #ccc;
  margin-bottom: 10px;
}

.todo:hover {
  background-color: #f5f5f5;
}


.todo.completed {
  text-decoration: line-through;
  color: #999;
}

.todo.editing {
  background-color: #fff;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.view {
  display: flex;
  align-items: center;
}

.toggle {
  margin-right: 10px;
  cursor: pointer;
}

.label {
  cursor: pointer;
}

.destroy {
  cursor: pointer;
  float: right;
  margin-left: 10px;
  color: #ccc;
}

.edit {
  width: 100%;
  padding: 10px;
  border: none;
  outline: none;
  background-color: transparent;
  font-size: 18px;
}

/* Footer section */
.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 40px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.todo-count {
  font-size: 16px;
  color: #999;
}

.filters {
  list-style: none;
  margin: 0;
  padding: 0;
}

.filters li {
  display: inline-block;
  margin-right: 10px;
}

.filters a {
  text-decoration: none;
  color: #999;
}

.filters a.selected {
  color: #333;
}

.clear-completed {
  cursor: pointer;
  color: #ccc;
}

/* Media queries for responsive design */
@media (max-width: 768px) {
  .header,
  .main,
  .footer {
    padding: 20px;
  }
}


</style>

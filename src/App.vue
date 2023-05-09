<script setup>
import './assets/base.css'
import './assets/index.css'
import { ref, reactive, computed } from 'vue';

let id = 0;
const filter = ref('all');
const newTodo = ref('');
const beforeEditCache = ref('');
let editTodo = reactive({
  id: null,
});
const state = reactive({
  todos: []
});


const showTodo = computed(() => {
  return state.todos.filter(data => {
    if (filter.value === 'all') {
      return true
    } else if (filter.value === 'active') {
      return !data.completed
    } else if (filter.value === 'completed') {
      return data.completed
    }
  })
})

function addTodo() {
  if (newTodo.value.trim()) {
    state.todos.push({
      id: id++,
      title: newTodo.value.trim(),
      completed: false,
    })
    newTodo.value = '';
    console.log(state.todos)
  }
}

const allDone = computed({
  get: function () {
    return state.todos.every(data => data.completed);
  },
  set: function (value) {
    state.todos.forEach(data => data.completed = value)
  }
})

function changeFilter(value) {
  filter.value = value
}

function editingMode(todo) {
  beforeEditCache.value = todo.title;
  editTodo.id = todo.id;
}

function removeTodo(todo) {
  let index = state.todos.indexOf(todo)
  state.todos.splice(index, 1)
}

const vTodoFocus = {
  updated: (el, binding) => {
    if (binding.value) {
      el.focus()
    }
  }
}

function doneEdit(todo) {
  if (todo.title.trim()) {
    editTodo.id = null;
    beforeEditCache.value = '';
  } else {
    alert('todo不能为空')
  }
}

function cancelEdit(todo) {
  todo.title = beforeEditCache.value;
  editTodo = {
    id: null,
    title: null,
  };
  beforeEditCache.value = '';
}

const removeCompleted = () => {
  state.todos = state.todos.filter(data => !data.completed)
}

</script>

<template>
  <div id="app">
    <section class="todoapp" v-cloak>
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo"
          @keyup.enter="addTodo">
      </header>
      <section class="main">
        <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone">
        <label for="toggle-all">Mark all as complete</label>
        <ul class="todo-list">
          <li :class="['todo', { completed: todo.completed === true }, { editing: editTodo.id === todo.id }]"
            v-for="(todo, index) in showTodo" :key="index">
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label @dblclick="editingMode(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input class="edit" type="text" v-todo-focus="todo.id == editTodo.id" v-model="todo.title"
              @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
          </li>
        </ul>
      </section>
      <footer class="footer">
        <span class="todo-count">
          <strong>{{ state.todos.length }}</strong> todo
        </span>
        <ul class="filters">
          <li><a href="#/all" :class="{ selected: filter === 'all' }" @click="changeFilter('all')">All</a></li>
          <li><a href="#/active" :class="{ selected: filter === 'active' }" @click="changeFilter('active')">Active</a>
          </li>
          <li><a href="#/completed" :class="{ selected: filter === 'completed' }"
              @click="changeFilter('completed')">Completed</a></li>
        </ul>
        <button class="clear-completed" v-show="state.todos.some(data => data.completed)" @click="removeCompleted">Clear
          completed</button>
      </footer>
    </section>
  </div>
</template>
<style scoped></style>

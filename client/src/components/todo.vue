<template>
  <div class="todoapp" v-cloak>
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length">
      <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone">
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li class="todo" v-for="todo in filteredTodos" :key="todo.id" :class="{completed: todo.completed, editing: todo == editedTodo}">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{todo.title}}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length">
      <span class="todo-count">
        <strong v-text="remaining"></strong> {{pluralize('item', remaining)}} left
      </span>
      <ul class="filters">
        <li><a @click="switchVisibility('all')" :class="{selected: visibility == 'all'}">All</a></li>
        <li><a @click="switchVisibility('active')" :class="{selected: visibility == 'active'}">Active</a></li>
        <li><a @click="switchVisibility('completed')" :class="{selected: visibility == 'completed'}">Completed</a></li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
        Clear completed
      </button>
    </footer>
  </div>
</template>

<script lang="ts">
interface Todo {
  id: number;
  title: null | string;
  completed: boolean;
}

interface Filters {
  [key: string]: (arg: Todo[]) => Todo[];
}

const filters: Filters = {
  all: (todos: Todo[]) => todos,
  active: (todos: Todo[]) => todos.filter((item: Todo) => !item.completed),
  completed: (todos: Todo[]) => todos.filter((item: Todo) => item.completed),
};

import { Component, Prop, Vue } from 'vue-property-decorator';

let beforeEditCache: null | string = null;

Vue.directive('todo-focus', (el, binding) => {
  if (binding.value) {
    el.focus();
  }
});

@Component
class TodoApp extends Vue {
  public visibility: string = 'all';
  public todos: Todo[] = [];
  public newTodo: string = '';
  public editedTodo: null | Todo = null;

  get filteredTodos() {
    return filters[this.visibility](this.todos);
  }

  get remaining() {
    return filters.active(this.todos).length;
  }

  get allDone() {
    return this.remaining === 0;
  }

  set allDone(value: boolean) {
    this.todos.forEach((v: Todo) => { v.completed = true; });
  }

  public switchVisibility(visibility: string) {
    this.visibility = visibility;
  }

  public pluralize(word: string, count: number) {
    return `${word}${count === 1 ? '' : 's'}`;
  }

  public addTodo() {
    const val = this.newTodo && this.newTodo.trim();

    this.todos.push(
      { id: this.todos.length - 1, title: val, completed: false } as Todo,
    );

    this.newTodo = '';
  }

  public removeTodo(payload: Todo) {
    const index = this.todos.indexOf(payload);
    this.todos.splice(index, 1);
  }

  public editTodo(payload: Todo) {
    beforeEditCache = payload.title;
    this.editedTodo = payload;
  }

  public cancelEdit(payload: Todo) {
    this.editedTodo = null;
    payload.title = beforeEditCache;
  }

  public doneEdit(payload: Todo) {
    if (this.editedTodo) { return; }

    this.editedTodo = null;
    payload.title = payload.title && payload.title.trim();

    if (!payload.title) {
      this.removeTodo(payload);
    }
  }

  public removeCompleted() {
    this.todos = filters.active(this.todos);
  }
}

export default TodoApp;
</script>

<style lang="less" scoped>

</style>

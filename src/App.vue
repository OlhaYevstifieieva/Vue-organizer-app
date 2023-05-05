<script>
import StatusFilter from './components/StatusFilter.vue';
import TodoItems from './components/TodoItem.vue';
import ErrorMessage from './components/ErrorMessage.vue'
import { getTodos, createTodos, updateTodos, removeTodos } from './data/todos';

export default{
  components: {
    StatusFilter,
    TodoItems,
    ErrorMessage,
},
data() {
  return {
    todos: [],
    title: '',
    status: 'all',
    errorMessage: '',
  }
},
mounted() {
  getTodos().then(res => {
    this.todos = res.data
  }).catch(() => {
    this.errorMessage = 'Unable to load todos'
  })
},
computed: {
 activeTodos() {
  return this.todos.filter(todo => !todo.completed);
 },
 completedTodos() {
  return this.todos.filter(todo => todo.completed);
 },
 // eslint-disable-next-line vue/return-in-computed-property
 visibleTodos() {
  switch(this.status) {
    case 'active':
      return this.activeTodos;
      case 'completed':
        return this.completedTodos;
    default:
      return this.todos;
  }
 },
},
methods: {
  handleSubmit() {
    createTodos(this.title).then((res) => {
        this.todos = [...this.todos, res.data]
        this.title = '';
      })
  },
  updateTodo({ id, title, completed }) {
    updateTodos({ id, title, completed }).then(res => {
      this.todos = this.todos.map(todo => todo.id !== id ? todo : res.data)
    })
  },
  deleteTodo(todoId) {
    removeTodos(todoId).then(() => {
      this.todos = this.todos.filter(todo => todo.id !== todoId)
    })
  },
  deleteCompletedTodos() {
    const todos = getTodos().then(res => {
      res.data.filter(r => {
        if (r.completed === true) {
          removeTodos(r.id).then(() => {
            this.todos = this.todos.filter(todo => todo.id !== r.id)
            })
        };
        return todos;
      });
    });
  },
}
}
</script>
<template>
   <div class="todoapp">
      <h1 class="todoapp__title">todos</h1>

      <div class="todoapp__content">
        <header class="todoapp__header">
          <button type="button" class="todoapp__toggle-all" :class="{active : activeTodos.length === 0 && todos.length !==0}">
            </button>
          <form @submit.prevent="handleSubmit">
            <input
              type="text"
              class="todoapp__new-todo"
              placeholder="What needs to be done?"
              v-model="title"
            />
          </form>
        </header>

        <TransitionGroup name="list" tag="section" class="todoapp__main">
          <TodoItems  v-for="todo, index of visibleTodos"
            :key="todo.id"
            :todo="todo"
            :index="index"
            @update="updateTodo"
            @delete="deleteTodo(todo.id)"
            />
        </TransitionGroup>
        <footer class="todoapp__footer">
          <span class="todo-count">
            {{ activeTodos.length }} items left
          </span>
          <StatusFilter  v-model="status"/>

          <button type="button" class="todoapp__clear-completed" v-if="completedTodos.length > 0" @click="deleteCompletedTodos">
            Clear completed
          </button>
        </footer>
      </div>
      <ErrorMessage
        class="is-warning"
        :active="errorMessage !== ''"
        @hide="errorMessage = ''">

        <template #default="{ x }">
          <p>{{ errorMessage }} {{ x }}</p>
        </template>

        <template #header>
          <p>Error</p>
        </template>
      </ErrorMessage>
    </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>

<script>
import StatusFilter from './components/StatusFilter.vue';
import TodoItem from './components/TodoItem.vue';

export default {
  components: {
    StatusFilter,
    TodoItem,
  },
  data() {
    return {
      todos: JSON.parse(localStorage.getItem('todos')) || [],
      title: '',
      status: 'all',
    };
  },
  computed: {
    activeTodos() {
      return this.todos.filter((todo) => !todo.completed);
    },
    completedTodos() {
      return this.todos.filter((todo) => todo.completed);
    },
    visibleTodos() {
      switch (this.status) {
        case 'active':
          return this.activeTodos;

        case 'completed':
          return this.completedTodos;

        default:
          return this.todos;
      }
    },
  },
  watch: {
    todos: {
      deep: true,
      handler() {
        localStorage.setItem('todos', JSON.stringify(this.todos));
      },
    },
  },
  methods: {
    handleSubmit() {
      if (this.title.trim() === '') {
        const errorMessage = document.querySelector('.message');
        errorMessage.classList.remove('message--hidden');
        return;
      }

      const newTodo = {
        id: Date.now(),
        title: this.title,
        completed: false,
      };

      this.todos = [...this.todos, newTodo];
      this.title = '';
    },
    updateTodo(updatedTodo) {
      this.todos = this.todos.map((todo) => {
        if (todo.id === updatedTodo.id) {
          return { ...todo, ...updatedTodo };
        }
        return todo;
      });
    },
    deleteTodo(todo) {
      this.todos = this.todos.filter((item) => item.id !== todo.id);
    },
    hideErrorMessage() {
      const errorMessage = document.querySelector('.message');
      errorMessage.classList.add('message--hidden');
    },
  },
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <article class="message message--hidden">
      <div class="message__hide-btn" @click="hideErrorMessage">
        <img
          width="10"
          height="10"
          src="https://img.icons8.com/metro/26/000000/multiply.png"
          alt="delete-sign"
          class="message__hide-img"
        />
      </div>
      <div>You cannot add empty values</div>
    </article>

    <div class="todoapp__content">
      <header class="todoapp__header">
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
        <TodoItem
          v-for="todo of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @update="updateTodo"
          @delete="deleteTodo"
        />
      </TransitionGroup>

      <footer class="todoapp__footer" v-if="todos.length">
        <StatusFilter v-model="status" />
        <span>Completed: {{ completedTodos.length }}</span>
      </footer>
    </div>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>

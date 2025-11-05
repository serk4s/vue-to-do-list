<script setup>
import { ref, computed } from 'vue';
import TodoList from './components/TodoList.vue';
import AddTodoModal from './components/AddTodoModal.vue';
import SearchBar from './components/SearchBar.vue';
import ThemeToggle from './components/ThemeToggle.vue';

// --- СОСТОЯНИЕ ---
const todos = ref([
  { id: 2, text: 'Проверить переключение тем', isCompleted: false },
  { id: 3, text: 'Удалить старые файлы стилей', isCompleted: true }
]);
const isModalOpen = ref(false);
const searchQuery = ref('');
const currentTheme = ref('dark'); // По умолчанию ставим Dark, как в оригинале

let nextId = 4;

// --- COMPUTED СВОЙСТВА ---
const filteredTodos = computed(() => {
  const filtered = todos.value.filter(todo =>
      todo.text.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
  return filtered.sort((a, b) => a.isCompleted - b.isCompleted);
});

// --- МЕТОДЫ ---
const openModal = () => { isModalOpen.value = true; };
const closeModal = () => { isModalOpen.value = false; };

const addTodo = (text) => {
  if (!text.trim()) return;
  todos.value.push({
    id: nextId++,
    text: text,
    isCompleted: false
  });
  closeModal();
};

const deleteTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id);
};

const toggleTodoStatus = (id) => {
  const todo = todos.value.find(t => t.id === id);
  if (todo) {
    todo.isCompleted = !todo.isCompleted;
  }
};

const updateTodoText = ({ id, newText }) => {
  const todo = todos.value.find(t => t.id === id);
  if (todo) {
    todo.text = newText;
  }
};

const toggleTheme = () => {
  currentTheme.value = currentTheme.value === 'light' ? 'dark' : 'light';
  // Ключевая строка для работы стилей SASS-миксина:
  document.body.setAttribute('data-theme', currentTheme.value);
};

// Устанавливаем начальную тему при загрузке
document.body.setAttribute('data-theme', currentTheme.value);

</script>

<template>
  <div class="app-container">
    <header class="header">
      <h1>Minimalist To-Do</h1>
      <ThemeToggle :current-theme="currentTheme" @toggle-theme="toggleTheme" />
    </header>

    <div class="controls-bar">
      <SearchBar v-model:search-query="searchQuery" />

      <button @click="openModal" class="btn-primary">
        + ADD
      </button>
    </div>

    <TodoList
        :todos="filteredTodos"
        @delete-todo="deleteTodo"
        @toggle-status="toggleTodoStatus"
        @update-text="updateTodoText"
    />

    <AddTodoModal
        v-if="isModalOpen"
        @close="closeModal"
        @add-todo="addTodo"
    />
  </div>
</template>

<style lang="sass">
// Используем @use для подключения основного файла стилей
@use './assets/main.scss' as *

.header
  display: flex
  justify-content: space-between
  align-items: center
  margin-bottom: 20px
</style>
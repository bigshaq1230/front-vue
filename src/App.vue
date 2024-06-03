<script setup>
import TodoList from './components/TodoList.vue';
import LogIn from './components/LogIn.vue';
import { ref, computed } from 'vue';
let logged_in = ref(false)

const routes = {
  '/': TodoList,
  '/login': LogIn
};
const currentPath = ref(window.location.hash);

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash;
});

const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || '/'];
});
</script>

<template>
  <component :is="currentView" :state="logged_in" @test="logged_in = true"></component>
</template>

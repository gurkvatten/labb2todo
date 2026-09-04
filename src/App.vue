<script setup>
import { ref } from 'vue';

const tasks = ref([
  { id: 1, title: 'Exempeluppgift', project: 'Sidoprojekt A', status: 'todo', priority: 'medium' },
  { id: 2, title: 'Fixa layout', project: 'Sidoprojekt B', status: 'in-progress', priority: 'high' },
])
</script>

<template>
  <div class="app">
    <h1>Projects Todo</h1>
  </div>
</template>



<style scoped>
.app {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem;
}
</style>
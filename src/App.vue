<script setup>
import { ref, computed } from 'vue';

const tasks = ref([
  { id: 1, title: 'Exempeluppgift', project: 'Sidoprojekt A', status: 'todo', priority: 'medium' },
  { id: 2, title: 'Fixa layout', project: 'Sidoprojekt B', status: 'in-progress', priority: 'high' },
])

let nextId = 3

const newTask = ref({
  title: '',
  project: '',
  priority: 'medium',
})

const showSettings = ref(false)
const draggedTaskId = ref(null)

function addTask() {
  if (!newTask.value.title.trim()) return

  tasks.value.push({
    id: nextId++,
    title: newTask.value.title,
    project: newTask.value.project,
    priority: newTask.value.priority,
    status: 'todo',
  })

  newTask.value = { title: '', project: '', priority: 'medium' }
}

function removeTask(id) {
  tasks.value = tasks.value.filter(t => t.id !== id)
}

function onDragStart(taskId) {
  draggedTaskId.value = taskId
}

function onDrop(newStatus) {
  const task = tasks.value.find(t => t.id === draggedTaskId.value)
  if (task) {
    task.status = newStatus
  }
  draggedTaskId.value = null
}

const todoTasks = computed(() => tasks.value.filter(t => t.status === 'todo'))
const inProgressTasks = computed(() => tasks.value.filter(t => t.status === 'in-progress'))
const doneTasks = computed(() => tasks.value.filter(t => t.status === 'done'))
</script>

<template>
  <div class="app">
    <div class="header">
      <h1>Projects Todo</h1>
      <button @click="showSettings = !showSettings">⚙️</button>
    </div>

    <div v-if="showSettings" class="settings">
      <p>Antal uppgifter totalt: {{ tasks.length }}</p>
      <p>Klara: {{ doneTasks.length }}</p>
    </div>

    <form class="add-form" @submit.prevent="addTask">
      <input
        v-model="newTask.title"
        type="text"
        placeholder="Vad ska göras?"
        required
      />
      <input
        v-model="newTask.project"
        type="text"
        placeholder="Projekt"
      />
      <select v-model="newTask.priority">
        <option value="low">Låg</option>
        <option value="medium">Medium</option>
        <option value="high">Hög</option>
      </select>
      <button type="submit">Lägg till</button>
    </form>

    <div class="board">
      <div
        class="column"
        @dragover.prevent
        @drop="onDrop('todo')"
      >
        <h2>Todo</h2>
        <div
          v-for="task in todoTasks"
          :key="task.id"
          class="card"
          draggable="true"
          @dragstart="onDragStart(task.id)"
        >
          <p class="title">{{ task.title }}</p>
          <p class="project">{{ task.project }}</p>
          <button class="remove-btn" @click="removeTask(task.id)">✕</button>
        </div>
      </div>
      <div
        class="column"
        @dragover.prevent
        @drop="onDrop('in-progress')"
      >
        <h2>In Progress</h2>
        <div
          v-for="task in inProgressTasks"
          :key="task.id"
          class="card"
          draggable="true"
          @dragstart="onDragStart(task.id)"
        >
          <p class="title">{{ task.title }}</p>
          <p class="project">{{ task.project }}</p>
          <button class="remove-btn" @click="removeTask(task.id)">✕</button>
        </div>
      </div>
      <div
        class="column"
        @dragover.prevent
        @drop="onDrop('done')"
      >
        <h2>Done</h2>
        <div
          v-for="task in doneTasks"
          :key="task.id"
          class="card"
          draggable="true"
          @dragstart="onDragStart(task.id)"
        >
          <p class="title">{{ task.title }}</p>
          <p class="project">{{ task.project }}</p>
          <button class="remove-btn" @click="removeTask(task.id)">✕</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem;
}
</style>
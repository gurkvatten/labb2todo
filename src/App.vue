<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const tasks = ref([
  { id: 1, title: 'Exempeluppgift', project: 'Sidoprojekt A', status: 'todo', priority: 'medium' },
  { id: 2, title: 'Fixa layout', project: 'Sidoprojekt B', status: 'in-progress', priority: 'high' },
  { id: 3, title: 'Skriv dokumentation', project: 'Sidoprojekt C', status: 'done', priority: 'low' },
])

let nextId = 4

const newTask = ref({
  title: '',
  project: '',
  priority: 'medium',
})

const showSettings = ref(false)
const draggedTaskId = ref(null)
const isDark = ref(false)
const userName = ref(localStorage.getItem('userName') || 'Johan')

onMounted(() => {
  const savedTasks = localStorage.getItem('tasks')
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks)
    nextId = tasks.value.length ? Math.max(...tasks.value.map(t => t.id)) + 1 : 1
  }

  const savedTheme = localStorage.getItem('theme')
  isDark.value = savedTheme === 'dark'
  document.documentElement.setAttribute('data-theme', isDark.value ? 'dark' : 'light')
})

watch(tasks, (newTasks) => {
  localStorage.setItem('tasks', JSON.stringify(newTasks))
}, { deep: true })

watch(isDark, (value) => {
  localStorage.setItem('theme', value ? 'dark' : 'light')
  document.documentElement.setAttribute('data-theme', value ? 'dark' : 'light')
})

watch(userName, (value) => {
  localStorage.setItem('userName', value)
})

const selectedProject = ref('all')

const uniqueProjects = computed(() => {
  const projects = tasks.value.map(t => t.project).filter(p => p.trim() !== '')
  return [...new Set(projects)]
})

const filteredTasks = computed(() => {
  if (selectedProject.value === 'all') {
    return tasks.value
  }
  return tasks.value.filter(t => t.project === selectedProject.value)
})
const filteredTodoTasks = computed(() => filteredTasks.value.filter(t => t.status === 'todo'))
const filteredInProgressTasks = computed(() => filteredTasks.value.filter(t => t.status === 'in-progress'))
const filteredDoneTasks = computed(() => filteredTasks.value.filter(t => t.status === 'done'))

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

function priorityColor(priority) {
  if (priority === 'high') return 'var(--danger-color)'
  if (priority === 'medium') return 'var(--warning-color)'
  return 'var(--success-color)'
}

const todoTasks = computed(() => tasks.value.filter(t => t.status === 'todo'))
const inProgressTasks = computed(() => tasks.value.filter(t => t.status === 'in-progress'))
const doneTasks = computed(() => tasks.value.filter(t => t.status === 'done'))
</script>

<template>
  <div class="app">
    <div class="header">
      <div>
        <h1>{{ userName }}s Kanban board</h1>
        <input v-model="userName" placeholder="Namn" class="name-input" />
      </div>
      <div class="header-actions">
        <button @click="isDark = !isDark" class="icon-btn">{{ isDark ? '☀️' : '🌙' }}</button>
        <button @click="showSettings = !showSettings" class="icon-btn">⚙️</button>
      </div>
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
        placeholder="Vilket projekt?"
      />
      <select v-model="newTask.priority">
        <option value="low">Låg</option>
        <option value="medium">Medium</option>
        <option value="high">Hög</option>
      </select>
      <button type="submit">Lägg till</button>
    </form>

    <div class="project-filter">
      <label for="project-select">Filtrera projekt:</label>
      <select id="project-select" v-model="selectedProject">
        <option value="all">Alla</option>
        <option v-for="project in uniqueProjects" :key="project" :value="project">{{ project }}</option>
      </select>
    </div>

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
          :style="{ borderLeft: '4px solid ' + priorityColor(task.priority) }"
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
          :style="{ borderLeft: '4px solid ' + priorityColor(task.priority) }"
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
          :style="{ borderLeft: '4px solid ' + priorityColor(task.priority) }"
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
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const DAY_MS = 1000 * 60 * 60 * 24

const tasks = ref([
  { id: 1, title: 'Exempeluppgift', project: 'Sidoprojekt A', status: 'todo', priority: 'medium', updatedAt: Date.now() - 1 * DAY_MS },
  { id: 2, title: 'Fixa layout', project: 'Sidoprojekt B', status: 'in-progress', priority: 'high', updatedAt: Date.now() - 5 * DAY_MS },
  { id: 3, title: 'Skriv dokumentation', project: 'Sidoprojekt C', status: 'done', priority: 'low', updatedAt: Date.now() - 20 * DAY_MS },
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
  const projects = tasks.value.map(t => t.project).filter(p => p && p.trim() !== '')
  return [...new Set(projects)]
})

const filteredTasks = computed(() => {
  if (selectedProject.value === 'all') {
    return tasks.value
  }
  return tasks.value.filter(t => t.project === selectedProject.value)
})

const todoTasks = computed(() => filteredTasks.value.filter(t => t.status === 'todo'))
const inProgressTasks = computed(() => filteredTasks.value.filter(t => t.status === 'in-progress'))
const doneTasks = computed(() => filteredTasks.value.filter(t => t.status === 'done'))


function daysSince(timestamp) {
  if (!timestamp) return 0
  return Math.floor((Date.now() - timestamp) / DAY_MS)
}

function opacityFromDays(days) {
  return Math.max(0.4, 1 - days * 0.04)
}

const projectActivity = computed(() => {
  return uniqueProjects.value
    .map(project => {
      const projectTasks = tasks.value.filter(t => t.project === project)
      const lastUpdated = Math.max(...projectTasks.map(t => t.updatedAt || 0))
      return { project, days: daysSince(lastUpdated) }
    })
    .sort((a, b) => b.days - a.days)
})

function projectDays(project) {
  const entry = projectActivity.value.find(p => p.project === project)
  return entry ? entry.days : 0
}

function cardStyle(task) {
  return {
    borderLeft: '4px solid ' + priorityColor(task.priority),
    opacity: opacityFromDays(daysSince(task.updatedAt)),
  }
}


function addTask() {
  if (!newTask.value.title.trim()) return

  tasks.value.push({
    id: nextId++,
    title: newTask.value.title,
    project: newTask.value.project,
    priority: newTask.value.priority,
    status: 'todo',
    updatedAt: Date.now(),
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
    task.updatedAt = Date.now()
  }
  draggedTaskId.value = null
}

function priorityColor(priority) {
  if (priority === 'high') return 'var(--danger-color)'
  if (priority === 'medium') return 'var(--warning-color)'
  return 'var(--success-color)'
}
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

      <div class="stale-projects">
        <p class="stale-heading">Mest bortglömda projekt:</p>
        <p
          v-for="item in projectActivity"
          :key="item.project"
          :style="{ opacity: opacityFromDays(item.days) }"
        >
          {{ item.project }} — {{ item.days }} dagar sen
        </p>
      </div>
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
        <option v-for="project in uniqueProjects" :key="project" :value="project">
          {{ project }} ({{ projectDays(project) }}d sen)
        </option>
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
          :style="cardStyle(task)"
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
          :style="cardStyle(task)"
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
          :style="cardStyle(task)"
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
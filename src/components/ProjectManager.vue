<!-- ProjectManager.vue -->
<template>
  <div class="project-manager">
    <h2>Управление проектами</h2>

    <div class="project-actions">
      <button @click="createNewProject" class="btn btn-primary">Создать новый проект</button>
      <button @click="openProjectList" class="btn btn-secondary">Открыть проект</button>
    </div>

    <div v-if="showProjectList" class="project-list">
      <h3>Сохраненные проекты</h3>
      <div class="projects">
        <div
          v-for="project in projects"
          :key="project.id"
          class="project-item"
          @click="loadProject(project)"
        >
          <div class="project-info">
            <h4>{{ project.name }}</h4>
            <p>{{ project.description }}</p>
            <small>{{ formatDate(project.updatedAt) }}</small>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showNewProjectForm" class="new-project-form">
      <h3>Новый проект</h3>
      <form @submit.prevent="saveNewProject">
        <div class="form-group">
          <label>Название проекта:</label>
          <input v-model="newProject.name" type="text" class="form-control" required />
        </div>
        <div class="form-group">
          <label>Описание:</label>
          <textarea v-model="newProject.description" class="form-control" rows="3"></textarea>
        </div>
        <div class="form-actions">
          <button type="submit" class="btn btn-primary">Создать</button>
          <button type="button" @click="cancelNewProject" class="btn btn-secondary">Отмена</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const showProjectList = ref(false)
const showNewProjectForm = ref(false)
const projects = ref([])
const newProject = ref({
  name: '',
  description: '',
})

// Загрузка списка проектов
const loadProjects = () => {
  const savedProjects = localStorage.getItem('grapesjs-projects')
  if (savedProjects) {
    projects.value = JSON.parse(savedProjects)
  }
}

// Создание нового проекта
const createNewProject = () => {
  showNewProjectForm.value = true
  showProjectList.value = false
  newProject.value = {
    name: '',
    description: '',
  }
}

// Открытие списка проектов
const openProjectList = () => {
  showProjectList.value = true
  showNewProjectForm.value = false
  loadProjects()
}

// Сохранение нового проекта
const saveNewProject = () => {
  const project = {
    id: Date.now(),
    name: newProject.value.name,
    description: newProject.value.description,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    data: null,
  }

  projects.value.push(project)
  localStorage.setItem('grapesjs-projects', JSON.stringify(projects.value))

  showNewProjectForm.value = false
  emit('project-created', project)
}

// Загрузка проекта
const loadProject = (project) => {
  emit('project-loaded', project)
  showProjectList.value = false
}

// Отмена создания проекта
const cancelNewProject = () => {
  showNewProjectForm.value = false
}

// Форматирование даты
const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('ru-RU')
}

const emit = defineEmits(['project-created', 'project-loaded'])

onMounted(() => {
  loadProjects()
})
</script>

<style scoped>
.project-manager {
  padding: 20px;
}

.project-actions {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
}

.project-list {
  margin-top: 20px;
}

.projects {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 15px;
  margin-top: 15px;
}

.project-item {
  border: 1px solid #dee2e6;
  border-radius: 8px;
  padding: 15px;
  cursor: pointer;
  transition: all 0.2s;
}

.project-item:hover {
  border-color: #007bff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.project-info h4 {
  margin: 0 0 5px 0;
  color: #333;
}

.project-info p {
  margin: 5px 0;
  color: #666;
  font-size: 14px;
}

.project-info small {
  color: #999;
}

.new-project-form {
  margin-top: 20px;
  max-width: 500px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-control {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.form-control:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.form-actions {
  display: flex;
  gap: 10px;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}
.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn:hover {
  opacity: 0.8;
}
</style>

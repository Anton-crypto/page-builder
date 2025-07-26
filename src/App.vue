<!-- App.vue -->
<template>
  <div id="app">
    <header class="app-header">
      <h1>Vue 3 + GrapesJS Builder</h1>
      <nav class="app-nav">
        <button
          @click="activeView = 'editor'"
          :class="{ active: activeView === 'editor' }"
          class="nav-btn"
        >
          Редактор
        </button>
        <button
          @click="activeView = 'projects'"
          :class="{ active: activeView === 'projects' }"
          class="nav-btn"
        >
          Проекты
        </button>
      </nav>
    </header>

    <main class="app-main">
      <GrapesJSEditor v-if="activeView === 'editor'" @project-saved="handleProjectSaved" />
      <ProjectManager
        v-if="activeView === 'projects'"
        @project-created="handleProjectCreated"
        @project-loaded="handleProjectLoaded"
      />
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import GrapesJSEditor from './components/GrapesJSEditor.vue'
import ProjectManager from './components/ProjectManager.vue'

const activeView = ref('editor')

const handleProjectSaved = (project) => {
  console.log('Проект сохранен:', project)
}

const handleProjectCreated = (project) => {
  console.log('Проект создан:', project)
  activeView.value = 'editor'
}

const handleProjectLoaded = (project) => {
  console.log('Проект загружен:', project)
  activeView.value = 'editor'
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background-color: #343a40;
  color: white;
  padding: 0 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.app-header h1 {
  font-size: 1.5rem;
  font-weight: normal;
}

.app-nav {
  display: flex;
  gap: 10px;
}

.nav-btn {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  padding: 15px 10px;
  cursor: pointer;
  font-size: 14px;
  transition: color 0.2s;
}

.nav-btn:hover {
  color: white;
}

.nav-btn.active {
  color: white;
  border-bottom: 2px solid white;
}

.app-main {
  flex: 1;
  overflow: hidden;
}

/* Font Awesome Icons (если используете) */
.fas {
  font-family: 'Font Awesome 5 Free';
  font-weight: 900;
}
</style>

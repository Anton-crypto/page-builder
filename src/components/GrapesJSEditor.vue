<template>
  <div class="grapesjs-editor">
    <!-- Панель инструментов -->
    <div class="toolbar">
      <div class="toolbar-left">
        <button @click="saveProject" class="btn btn-primary">
          <i class="fas fa-save"></i> Сохранить
        </button>
        <button @click="previewMode" class="btn btn-info">
          <i class="fas fa-eye"></i> Предпросмотр
        </button>
      </div>
      <div class="toolbar-right">
        <button @click="clearEditor" class="btn btn-danger">
          <i class="fas fa-trash"></i> Очистить
        </button>
      </div>
    </div>
    <!-- Основная область редактора -->
    <div class="editor-wrapper">
      <!-- Панель блоков -->
      <div class="panel panel-left" ref="blocksPanel">
        <div class="panel-header">
          <h3>Блоки</h3>
        </div>
        <div id="blocks" class="panel-content"></div>
      </div>
      <!-- Редактор -->
      <div class="editor-main">
        <div ref="editorContainer" class="editor-container"></div>
      </div>
      <!-- Панель стилей -->
      <div class="panel panel-right" ref="stylesPanel">
        <div class="panel-header">
          <h3>Стили</h3>
        </div>
        <div id="styles" class="panel-content"></div>
      </div>
    </div>
    <!-- Модальное окно для предпросмотра -->
    <div v-if="showPreview" class="preview-modal" @click="closePreview">
      <div class="preview-content" @click.stop>
        <div class="preview-header">
          <h2>Предпросмотр</h2>
          <button @click="closePreview" class="close-btn">&times;</button>
        </div>
        <div class="preview-body" v-html="previewContent"></div>
      </div>
    </div>
    <!-- Статус бар -->
    <div class="status-bar">
      <div class="status-item">
        <span>Статус: {{ editorStatus }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import grapesjs from 'grapesjs'
import 'grapesjs/dist/css/grapes.min.css'

// Ссылки на элементы
const editorContainer = ref(null)
const blocksPanel = ref(null)
const stylesPanel = ref(null)
const previewFrame = ref(null)

// Состояния
const showPreview = ref(false)
const editorStatus = ref('Загрузка...')
const editorInstance = ref(null)

// Данные для предпросмотра
const previewContent = computed(() => {
  if (!editorInstance.value) return ''
  const html = editorInstance.value.getHtml()
  const css = editorInstance.value.getCss()
  return `<style>${css}</style>${html}`
})

// Плагин для колонок
const columnPlugin = (editor) => {
  // ... (ваш существующий код для columnPlugin)
  // Добавление кастомных блоков колонок
  editor.BlockManager.add('columns-2', {
    label: '2 Колонки',
    category: 'Колонки',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <rect x="2" y="2" width="16" height="26" fill="#ccc"/>
        <rect x="22" y="2" width="16" height="26" fill="#ccc"/>
      </svg>
    `,
    content: `
      <div class="row" style="display: flex; gap: 20px; margin: 20px 0;">
        <div class="column" style="flex: 1; min-height: 100px; border: 1px dashed #ccc; padding: 15px;">
          <p>Колонка 1</p>
        </div>
        <div class="column" style="flex: 1; min-height: 100px; border: 1px dashed #ccc; padding: 15px;">
          <p>Колонка 2</p>
        </div>
      </div>
    `,
  })

  editor.BlockManager.add('columns-3', {
    label: '3 Колонки',
    category: 'Колонки',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <rect x="1" y="2" width="11" height="26" fill="#ccc"/>
        <rect x="14" y="2" width="11" height="26" fill="#ccc"/>
        <rect x="27" y="2" width="11" height="26" fill="#ccc"/>
      </svg>
    `,
    content: `
      <div class="row" style="display: flex; gap: 15px; margin: 20px 0;">
        <div class="column" style="flex: 1; min-height: 100px; border: 1px dashed #ccc; padding: 15px;">
          <p>Колонка 1</p>
        </div>
        <div class="column" style="flex: 1; min-height: 100px; border: 1px dashed #ccc; padding: 15px;">
          <p>Колонка 2</p>
        </div>
        <div class="column" style="flex: 1; min-height: 100px; border: 1px dashed #ccc; padding: 15px;">
          <p>Колонка 3</p>
        </div>
      </div>
    `,
  })

  // Базовые блоки
  editor.BlockManager.add('text', {
    label: 'Текст',
    category: 'Базовые',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <text x="5" y="20" font-family="Arial" font-size="12" fill="#444">Текст</text>
      </svg>
    `,
    content: '<p>Ваш текст здесь</p>',
  })

  editor.BlockManager.add('image', {
    label: 'Изображение',
    category: 'Базовые',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <rect x="2" y="2" width="36" height="26" fill="#ddd" stroke="#999"/>
        <circle cx="20" cy="15" r="6" fill="#999"/>
      </svg>
    `,
    content: {
      type: 'image',
      activeOnRender: 1,
      attributes: {
        src: 'https://via.placeholder.com/300x200?text=Изображение',
        style: 'max-width: 100%; height: auto;',
      },
    },
  })

  editor.BlockManager.add('heading', {
    label: 'Заголовок',
    category: 'Базовые',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <text x="5" y="18" font-family="Arial" font-size="16" font-weight="bold" fill="#444">H1</text>
      </svg>
    `,
    content: '<h1>Заголовок</h1>',
  })
}

// Новый плагин для сайдбара
const sidebarPlugin = (editor) => {
  // Блок "Сайдбар"
  editor.BlockManager.add('sidebar', {
    label: 'Сайдбар',
    category: 'Навигация',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <rect x="2" y="2" width="12" height="26" fill="#e0e0e0" stroke="#999"/>
        <rect x="16" y="2" width="22" height="4" fill="#bdbdbd"/>
        <rect x="16" y="8" width="18" height="3" fill="#bdbdbd"/>
        <rect x="16" y="13" width="20" height="3" fill="#bdbdbd"/>
      </svg>
    `,
    content: `
      <div class="sidebar-container" style="display: flex; min-height: 400px; font-family: Arial, sans-serif;">
        <!-- Сайдбар -->
        <div class="sidebar" style="width: 250px; background-color: #f5f5f5; padding: 20px; border-right: 1px solid #ddd;">
          <!-- Поиск -->
          <div class="sidebar-search" style="margin-bottom: 20px;">
            <input type="text" placeholder="Поиск по разделу..." style="width: 100%; padding: 8px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box;">
          </div>
          <!-- Навигация -->
          <nav class="sidebar-nav">
            <ul style="list-style: none; padding: 0; margin: 0;">
              <li style="margin-bottom: 10px;"><a href="#" style="text-decoration: none; color: #333; display: block; padding: 5px 0;">Главная</a></li>
              <li style="margin-bottom: 10px;">
                <a href="#" style="text-decoration: none; color: #333; display: block; padding: 5px 0;">Продукты</a>
                <ul style="list-style: none; padding-left: 15px; margin-top: 5px;">
                  <li style="margin-bottom: 5px;"><a href="#" style="text-decoration: none; color: #666; font-size: 0.9em;">Категория 1</a></li>
                  <li style="margin-bottom: 5px;"><a href="#" style="text-decoration: none; color: #666; font-size: 0.9em;">Категория 2</a></li>
                </ul>
              </li>
              <li style="margin-bottom: 10px;"><a href="#" style="text-decoration: none; color: #333; display: block; padding: 5px 0;">О нас</a></li>
              <li><a href="#" style="text-decoration: none; color: #333; display: block; padding: 5px 0;">Контакты</a></li>
            </ul>
          </nav>
        </div>
        <!-- Основной контент -->
        <div class="main-content" style="flex: 1; padding: 20px;">
          <h1>Добро пожаловать</h1>
          <p>Это основная область содержимого. Здесь будет отображаться информация, соответствующая выбранному разделу в сайдбаре.</p>
        </div>
      </div>
    `,
  });

  // Блок "Текстовая карточка"
  editor.BlockManager.add('text-card', {
    label: 'Текстовая карточка',
    category: 'Компоненты',
    media: `
      <svg width="40" height="30" viewBox="0 0 40 30">
        <rect x="2" y="2" width="36" height="26" fill="#e3f2fd" stroke="#90caf9"/>
        <rect x="5" y="5" width="30" height="10" fill="#bbdefb"/>
        <rect x="5" y="17" width="30" height="3" fill="#90caf9"/>
        <rect x="5" y="22" width="25" height="3" fill="#90caf9"/>
      </svg>
    `,
    content: `
      <div class="text-card" style="border: 1px solid #ddd; border-radius: 8px; padding: 15px; margin-bottom: 20px; background-color: #fff; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
        <img src="https://via.placeholder.com/300x150?text=Изображение" alt="Изображение карточки" style="width: 100%; height: auto; border-radius: 4px; margin-bottom: 10px;">
        <h3 style="margin-top: 0; margin-bottom: 10px; color: #333;">Заголовок карточки</h3>
        <p style="margin: 0; color: #666; font-size: 0.9em;">Краткое описание содержимого карточки. Может включать ключевые моменты или анонс материала.</p>
      </div>
    `,
  });
};

// Инициализация редактора
const initEditor = () => {
  try {
    const editor = grapesjs.init({
      container: editorContainer.value,
      height: '100%',
      width: '100%',
      showOffsets: true,
      noticeOnUnload: false,
      // Плагины
      plugins: [columnPlugin, sidebarPlugin], // Добавлен sidebarPlugin
      // Настройки блоков
      blockManager: {
        appendTo: '#blocks',
      },
      // Настройки стилей
      styleManager: {
        appendTo: '#styles',
        sectors: [
          {
            name: 'Размеры',
            open: true,
            buildProps: ['width', 'height', 'max-width', 'min-height', 'margin', 'padding'],
          },
          {
            name: 'Типографика',
            open: true,
            buildProps: [
              'font-family',
              'font-size',
              'font-weight',
              'color',
              'line-height',
              'text-align',
            ],
          },
          {
            name: 'Декорации',
            open: true,
            buildProps: ['background-color', 'border', 'border-radius', 'box-shadow'],
          },
        ],
      },
      // Настройки хранилища
      storageManager: {
        id: 'gjs-',
        type: 'local',
        autosave: true,
        autoload: true,
        stepsBeforeSave: 1,
        storeComponents: true,
        storeStyles: true,
        storeHtml: true,
        storeCss: true,
      },
      // Настройки устройства
      deviceManager: {
        devices: [
          {
            name: 'Десктоп',
            width: '',
          },
          {
            name: 'Планшет',
            width: '768px',
          },
          {
            name: 'Мобильный',
            width: '320px',
          },
        ],
      },
    })
    editorInstance.value = editor
    // Обработчики событий
    editor.on('load', () => {
      editorStatus.value = 'Готов'
    })
    editor.on('component:add', () => {
      editorStatus.value = 'Элемент добавлен'
      setTimeout(() => {
        editorStatus.value = 'Готов'
      }, 1000)
    })
    editorStatus.value = 'Готов'
  } catch (error) {
    editorStatus.value = 'Ошибка инициализации'
    console.error('Ошибка инициализации GrapesJS:', error)
  }
}

// Сохранение проекта
const saveProject = () => {
  if (editorInstance.value) {
    editorInstance.value.store()
    editorStatus.value = 'Проект сохранен'
    setTimeout(() => {
      editorStatus.value = 'Готов'
    }, 2000)
  }
}

// Режим предпросмотра
const previewMode = () => {
  showPreview.value = true
}

// Закрытие предпросмотра
const closePreview = () => {
  showPreview.value = false
}

// Очистка редактора
const clearEditor = () => {
  if (editorInstance.value && confirm('Вы уверены, что хотите очистить весь контент?')) {
    editorInstance.value.DomComponents.clear()
    editorStatus.value = 'Редактор очищен'
    setTimeout(() => {
      editorStatus.value = 'Готов'
    }, 2000)
  }
}

// Жизненный цикл
onMounted(() => {
  // Небольшая задержка для гарантии рендеринга DOM
  setTimeout(() => {
    initEditor()
  }, 100)
})

onUnmounted(() => {
  if (editorInstance.value) {
    editorInstance.value.destroy()
  }
})
</script>

<style scoped>
.grapesjs-editor {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: Arial, sans-serif;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #f8f9fa;
  border-bottom: 1px solid #dee2e6;
}

.toolbar-left,
.toolbar-right {
  display: flex;
  gap: 10px;
}

.btn {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
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

.btn-info {
  background-color: #17a2b8;
  color: white;
}

.btn-warning {
  background-color: #ffc107;
  color: black;
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn:hover {
  opacity: 0.8;
}

.editor-wrapper {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.panel {
  background-color: #ffffff;
  border: 1px solid #dee2e6;
  display: flex;
  flex-direction: column;
}

.panel-left {
  width: 250px;
  border-right: 1px solid #dee2e6;
}

.panel-right {
  width: 300px;
  border-left: 1px solid #dee2e6;
}

.panel-header {
  padding: 10px;
  background-color: #e9ecef;
  border-bottom: 1px solid #dee2e6;
}

.panel-header h3 {
  margin: 0;
  font-size: 14px;
  font-weight: bold;
}

.panel-content {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}

.editor-main {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.editor-container {
  flex: 1;
  overflow: hidden;
}

/* Стили для блоков */
.gjs-block {
  min-height: auto !important;
  padding: 10px !important;
  margin-bottom: 5px !important;
}

.gjs-block-label {
  font-size: 12px !important;
}

.gjs-two-color {
  background: #f8f9fa !important;
}

.gjs-four-color {
  background: #e9ecef !important;
}

.gjs-four-color-h:hover {
  background: #dee2e6 !important;
}

/* Предпросмотр */
.preview-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.preview-content {
  width: 90%;
  height: 90%;
  background-color: white;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
}

.preview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  border-bottom: 1px solid #dee2e6;
}

.preview-header h2 {
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
}

.preview-body {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.status-bar {
  padding: 5px 10px;
  background-color: #f8f9fa;
  border-top: 1px solid #dee2e6;
  font-size: 12px;
  color: #6c757d;
}

/* Категории блоков */
.gjs-block-category {
  margin-bottom: 10px;
}

.gjs-block-category .gjs-title {
  background-color: #e3f2fd !important;
  color: #1976d2 !important;
  font-weight: bold !important;
  padding: 8px !important;
}

/* Стили для колонок в редакторе */
.row {
  display: flex !important;
  gap: 20px !important;
  margin: 20px 0 !important;
}

.column {
  flex: 1 !important;
  min-height: 100px !important;
  border: 1px dashed #ccc !important;
  padding: 15px !important;
}
</style>
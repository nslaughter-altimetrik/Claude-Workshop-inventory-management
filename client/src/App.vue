<template>
  <div class="app-shell">
    <Sidebar
      :routes="navRoutes"
      :brand-name="t('nav.companyName')"
      :brand-subtitle="t('nav.subtitle')"
    >
      <template #footer>
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </template>
    </Sidebar>
    <main class="app-main">
      <FilterBar />
      <div class="app-main-inner">
        <router-view />
      </div>
    </main>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import Sidebar from './components/Sidebar.vue'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

const ICON_HOME = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M3 9.5L12 3l9 6.5V20a1 1 0 0 1-1 1h-5v-7h-6v7H4a1 1 0 0 1-1-1V9.5z"/></svg>'
const ICON_BOX = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 8L12 3 3 8v8l9 5 9-5V8z"/><path d="M3 8l9 5 9-5"/><path d="M12 13v8"/></svg>'
const ICON_CART = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="9" cy="20" r="1.5"/><circle cx="18" cy="20" r="1.5"/><path d="M2 3h2.5l3 12.5h12L22 7H6"/></svg>'
const ICON_REFRESH = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12a9 9 0 1 1-3.5-7.1"/><path d="M21 4v5h-5"/></svg>'
const ICON_DOLLAR = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="2" x2="12" y2="22"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>'
const ICON_TREND = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 17 9 11 13 15 21 7"/><polyline points="15 7 21 7 21 13"/></svg>'
const ICON_BARS = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><line x1="6" y1="20" x2="6" y2="12"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="18" y1="20" x2="18" y2="14"/><line x1="3" y1="20" x2="21" y2="20"/></svg>'

export default {
  name: 'App',
  components: {
    Sidebar,
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const navRoutes = computed(() => [
      { path: '/',           label: t('nav.overview'),       icon: ICON_HOME },
      { path: '/inventory',  label: t('nav.inventory'),      icon: ICON_BOX },
      { path: '/orders',     label: t('nav.orders'),         icon: ICON_CART },
      { path: '/restocking', label: t('nav.restocking'),     icon: ICON_REFRESH },
      { path: '/spending',   label: t('nav.finance'),        icon: ICON_DOLLAR },
      { path: '/demand',     label: t('nav.demandForecast'), icon: ICON_TREND },
      { path: '/reports',    label: 'Reports',               icon: ICON_BARS }
    ])

    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      navRoutes,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
/* Linear-style design tokens — added by vue-saas-redesign */
:root {
  --color-bg: #fafafa;
  --color-surface: #ffffff;
  --color-sidebar: #fbfbfa;
  --color-hover: #f4f4f3;
  --color-border: #e8e8e6;
  --color-border-strong: #d4d4d2;
  --color-ink: #18181b;
  --color-ink-soft: #52525b;
  --color-muted: #71717a;
  --color-faint: #a1a1aa;
  --color-accent: #5e6ad2;
  --color-accent-soft: #eeeefe;
  --color-accent-ink: #4c54b8;
  --color-success: #10b981;
  --color-success-soft: #d1fae5;
  --color-warning: #f59e0b;
  --color-warning-soft: #fef3c7;
  --color-danger: #ef4444;
  --color-danger-soft: #fee2e2;
  --color-info: #3b82f6;
  --color-info-soft: #dbeafe;
  --space-1: 4px; --space-2: 8px; --space-3: 12px; --space-4: 16px;
  --space-5: 20px; --space-6: 24px; --space-8: 32px; --space-10: 40px;
  --space-12: 48px; --space-16: 64px;
  --radius-sm: 4px; --radius-md: 6px; --radius-lg: 8px; --radius-xl: 12px; --radius-full: 999px;
  --font-sans: -apple-system, BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  --font-mono: ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, monospace;
  --text-xs: 11px; --text-sm: 12px; --text-base: 13px; --text-md: 14px; --text-lg: 16px; --text-xl: 20px; --text-2xl: 24px;
  --leading-tight: 1.3; --leading-normal: 1.55; --leading-relaxed: 1.7;
  --shadow-xs: 0 1px 1px rgba(15, 23, 42, 0.03);
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04), 0 1px 3px rgba(15, 23, 42, 0.06);
  --shadow-md: 0 4px 6px -1px rgba(15, 23, 42, 0.08), 0 2px 4px -2px rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(15, 23, 42, 0.08), 0 4px 6px -4px rgba(15, 23, 42, 0.06);
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 56px;
  --content-max-width: 1280px;
  --header-height: 56px;
  --transition-fast: 100ms ease-out;
  --transition-base: 150ms ease-out;
  --transition-slow: 220ms ease-out;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  background: var(--color-bg);
  color: var(--color-ink);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app-shell {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
}

.app-main {
  display: flex;
  flex-direction: column;
  min-width: 0;
  background: var(--color-bg);
}

.app-main-inner {
  flex: 1;
  padding: var(--space-6) var(--space-8);
  max-width: var(--content-max-width);
  width: 100%;
  margin: 0 auto;
}

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: var(--text-xl);
  font-weight: 600;
  color: var(--color-ink);
  margin-bottom: var(--space-1);
  letter-spacing: -0.02em;
}

.page-header p {
  color: var(--color-muted);
  font-size: var(--text-md);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-4);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-4) var(--space-5);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-xs);
  transition: border-color var(--transition-fast), box-shadow var(--transition-fast);
}

.stat-card:hover {
  border-color: var(--color-border-strong);
  box-shadow: var(--shadow-sm);
}

.stat-label {
  color: var(--color-muted);
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: var(--text-2xl);
  font-weight: 700;
  color: var(--color-ink);
  letter-spacing: -0.02em;
}

.stat-card.warning .stat-value { color: var(--color-warning); }
.stat-card.success .stat-value { color: var(--color-success); }
.stat-card.danger  .stat-value { color: var(--color-danger);  }
.stat-card.info    .stat-value { color: var(--color-info);    }

.card {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-xs);
  margin-bottom: var(--space-5);
  overflow: hidden;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-4) var(--space-5);
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: var(--text-md);
  font-weight: 600;
  color: var(--color-ink);
  letter-spacing: -0.01em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-bg);
}

th {
  text-align: left;
  padding: var(--space-3) var(--space-4);
  font-weight: 600;
  color: var(--color-ink-soft);
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  border-bottom: 1px solid var(--color-border);
}

td {
  padding: var(--space-3) var(--space-4);
  border-top: 1px solid var(--color-border);
  color: var(--color-ink);
  font-size: var(--text-base);
}

tbody tr {
  transition: background-color var(--transition-fast);
}

tbody tr:hover {
  background: var(--color-hover);
}

.badge {
  display: inline-block;
  padding: 2px var(--space-2);
  border-radius: var(--radius-sm);
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.badge.success    { background: var(--color-success-soft); color: #065f46; }
.badge.warning    { background: var(--color-warning-soft); color: #92400e; }
.badge.danger     { background: var(--color-danger-soft);  color: #991b1b; }
.badge.info       { background: var(--color-info-soft);    color: #1e40af; }
.badge.increasing { background: var(--color-success-soft); color: #065f46; }
.badge.decreasing { background: var(--color-danger-soft);  color: #991b1b; }
.badge.stable     { background: var(--color-accent-soft);  color: var(--color-accent-ink); }
.badge.high       { background: var(--color-danger-soft);  color: #991b1b; }
.badge.medium     { background: var(--color-warning-soft); color: #92400e; }
.badge.low        { background: var(--color-info-soft);    color: #1e40af; }

.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--color-muted);
  font-size: var(--text-md);
}

.error {
  background: var(--color-danger-soft);
  border: 1px solid #fca5a5;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: var(--text-md);
}
</style>

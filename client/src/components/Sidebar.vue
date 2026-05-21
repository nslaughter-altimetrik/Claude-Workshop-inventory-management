<template>
  <aside class="sidebar">
    <div class="sidebar-brand">
      <div class="brand-mark" aria-hidden="true">{{ brandInitial }}</div>
      <div class="brand-text">
        <div class="brand-name">{{ brandName }}</div>
        <div v-if="brandSubtitle" class="brand-subtitle">{{ brandSubtitle }}</div>
      </div>
    </div>

    <nav class="sidebar-nav" aria-label="Primary">
      <router-link
        v-for="route in routes"
        :key="route.path"
        :to="route.path"
        class="nav-item"
        :class="{ active: $route.path === route.path }"
      >
        <span class="nav-icon" v-html="route.icon" aria-hidden="true"></span>
        <span class="nav-label">{{ route.label }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <slot name="footer" />
    </div>
  </aside>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'Sidebar',
  props: {
    routes: { type: Array, required: true },
    brandName: { type: String, default: 'App' },
    brandSubtitle: { type: String, default: '' }
  },
  setup(props) {
    const brandInitial = computed(() => (props.brandName || 'A').charAt(0).toUpperCase())
    return { brandInitial }
  }
}
</script>

<style scoped>
.sidebar {
  display: flex;
  flex-direction: column;
  background: var(--color-sidebar);
  border-right: 1px solid var(--color-border);
  height: 100vh;
  position: sticky;
  top: 0;
  font-family: var(--font-sans);
  font-size: var(--text-base);
  color: var(--color-ink-soft);
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.brand-mark {
  width: 32px;
  height: 32px;
  border-radius: var(--radius-md);
  background: var(--color-accent);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: var(--text-md);
  flex-shrink: 0;
  letter-spacing: -0.02em;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
  gap: 1px;
}

.brand-name {
  font-weight: 600;
  color: var(--color-ink);
  font-size: var(--text-md);
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.2;
}

.brand-subtitle {
  font-size: var(--text-xs);
  color: var(--color-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.3;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: var(--space-3) var(--space-2);
  gap: 2px;
  overflow-y: auto;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-md);
  color: var(--color-ink-soft);
  text-decoration: none;
  font-size: var(--text-base);
  font-weight: 500;
  transition: background var(--transition-fast), color var(--transition-fast);
}

.nav-item:hover {
  background: var(--color-hover);
  color: var(--color-ink);
}

.nav-item.active {
  background: var(--color-accent-soft);
  color: var(--color-accent-ink);
}

.nav-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.nav-icon :deep(svg) {
  width: 16px;
  height: 16px;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-footer {
  padding: var(--space-3);
  border-top: 1px solid var(--color-border);
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}
</style>

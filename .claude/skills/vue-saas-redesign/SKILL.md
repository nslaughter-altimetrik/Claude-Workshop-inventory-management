---
name: vue-saas-redesign
description: Redesign a Vue 3 application's top-nav UI into a modern SaaS-style interface with a vertical left sidebar, consistent 8-pt spacing, and Linear-inspired typography. Generic across Vue 3 + Vue Router projects. Use when the user asks to convert top nav to sidebar, wants a SaaS-style or Linear/Stripe/Vercel-like UI, asks to "polish" or "modernize" a Vue 3 frontend, or wants left-rail navigation.
---

# Vue 3 SaaS Redesign

Converts a Vue 3 + Vue Router app from a horizontal top-nav layout into a modern SaaS-style interface: vertical sidebar on the left, refined typography, an 8-pt spacing scale, and a Linear-inspired visual language. Generic — works on any Vue 3 codebase.

## When to use this skill

Invoke when the user says things like:
- "Convert the top nav to a sidebar"
- "Make this look like Linear / Stripe / Vercel"
- "SaaS-style redesign"
- "Polish / modernize the UI"
- "Move nav to the left rail"
- "Tighten up the spacing and typography"

Do **not** invoke for: pure visual tweaks (single-color change), component-level styling, or apps that aren't Vue 3.

## Inputs

The skill accepts these optional flags in the user prompt:

| Flag | Default | Meaning |
|---|---|---|
| `--style=linear\|stripe\|notion` | `linear` | Visual flavor. v1 implements `linear` only; other values fall back with a note. |
| `--width=240` | `240` | Sidebar width in px. Applied to `--sidebar-width`. |
| `--collapsible` | off | Adds a collapse toggle (v1 stub: documents the prop; full impl TODO). |

If no flag is passed, use defaults silently.

## Workflow

Run these phases in order. Phase 2 ends with an `AskUserQuestion` confirmation — **do not proceed to Phase 3 without explicit approval.**

---

### Phase 1 — Discovery (read-only)

Goal: build a structured picture of the target app's nav, routing, and existing design system.

1. **Verify the stack.** Read `package.json`. Required: `vue: ^3.x` and `vue-router: ^4.x` in dependencies. If Vue 2 or no Vue, **stop** and tell the user the skill doesn't support their stack.

2. **Locate the router config.** Try in order:
   - `src/router/index.js`
   - `src/router.js`
   - `src/main.js` (createRouter call)

   Extract every `{ path, component }` entry. Note the order.

3. **Locate the current nav.** Read `src/App.vue`. Find the nav block — heuristics:
   - A `<header>`, `<nav>`, or element with class matching `nav|header|top|tabs`.
   - Containing two or more `<router-link>` elements.
   - Capture: container element + class, child link templates (especially how they declare active state), label source (`{{ t('nav.X') }}` vs hard-coded string).

4. **Detect preserved components.** Grep `src/components/` and `src/composables/`:
   - `ProfileMenu`, `UserMenu`, `Avatar` → preserved into sidebar footer.
   - `LanguageSwitcher`, `LocaleSwitcher`, theme toggles → preserved into sidebar footer.
   - `FilterBar` or sub-nav components currently rendered at the App.vue level → preserved as a sub-header inside `<main>`.

5. **Detect existing tokens.** Look for CSS variables (`--color-*`, `--space-*`) in:
   - `src/App.vue` `<style>` blocks (scoped and unscoped).
   - `src/assets/*.css`, `src/styles/*.css`, `src/main.css`.
   If found, list them so Phase 3 can avoid overwriting collisions.

6. **Detect a `vue-expert` agent.** Check for `.claude/agents/vue-expert.md`. If present, all `.vue` writes in Phase 3 **must** be delegated via the `Agent` tool with `subagent_type: 'vue-expert'`. This is a project-level constraint (often pinned in `CLAUDE.md`).

Phase 1 output: a short structured summary block — route table, label source, preserved components, existing tokens (if any), vue-expert delegation flag.

---

### Phase 2 — Preview (no writes)

Goal: show the user what the change will look like and get explicit approval.

Produce three artifacts as a single message to the user:

**A. Before/after layout sketch** (ASCII or markdown):
```
BEFORE                              AFTER
┌──────────────────────────────┐    ┌──────┬────────────────────────┐
│ Brand   Tab Tab Tab Tab  User│    │      │ Page header            │
├──────────────────────────────┤    │ Logo │                        │
│ FilterBar (if present)       │    │      │ FilterBar (if present) │
├──────────────────────────────┤    │ Nav  │                        │
│                              │    │      │ Content                │
│ Content                      │    │      │                        │
│                              │    │ User │                        │
└──────────────────────────────┘    └──────┴────────────────────────┘
```

**B. File change plan** — exact list of touched files:
| File | Action |
|---|---|
| `src/components/Sidebar.vue` | Create |
| `src/styles/tokens.css` (or App.vue inline) | Create / add tokens |
| `src/App.vue` | Replace nav block; restructure to grid; reapply tokens |
| `src/main.js` | Add `import './styles/tokens.css'` if standalone file |

**C. Route → icon mapping** based on path keywords (case-insensitive, longest match wins):

| Path keyword | Icon |
|---|---|
| `/` (root) or `dashboard` or `overview` or `home` | Home |
| `inventory` or `stock` or `products` or `items` | Box |
| `orders` or `cart` or `checkout` | ShoppingCart |
| `restocking` or `replenish` or `reorder` | RefreshCw |
| `finance` or `spending` or `billing` or `revenue` | DollarSign |
| `demand` or `forecast` or `trend` or `growth` | TrendingUp |
| `reports` or `analytics` or `metrics` or `insights` | BarChart3 |
| `docs` or `notes` or `articles` | FileText |
| `settings` or `config` or `preferences` | Settings |
| `users` or `team` or `members` | Users |
| (anything else) | Square (fallback) |

The icon SVGs are pasted from `reference/icons.md`.

After printing the three artifacts, call `AskUserQuestion` with options like:
- "Apply as shown (Recommended)"
- "Apply with edits" — (then ask which routes/icons to adjust)
- "Cancel"

If the user picks Cancel, stop. If Edits, capture the diffs and re-render the plan, then re-confirm.

---

### Phase 3 — Apply

Goal: write the redesign with minimal disruption to logic. **Only enter this phase after explicit user approval in Phase 2.**

#### Step 1 — Tokens

If the project has any global stylesheet (`src/styles/main.css`, `src/assets/main.css`, etc.):
- Create `src/styles/tokens.css` with the contents of `reference/tokens.css`.
- Add `import './styles/tokens.css'` near the top of `src/main.js`.

If no global stylesheet exists:
- Inline the `:root { ... }` block from `reference/tokens.css` into the unscoped `<style>` of `App.vue` (one block, at the top of the unscoped styles).

If existing tokens were detected in Phase 1: **merge, don't overwrite.** Keep the project's existing variable names; only add new ones the design needs (e.g. `--sidebar-width`). Add a brief comment in the tokens block: `/* Added by vue-saas-redesign — merge with care */`.

#### Step 2 — Sidebar component

Create `src/components/Sidebar.vue` from the template in `reference/sidebar.vue.example`. Adjustments per project:
- If i18n was detected (`{{ t('nav.X') }}` pattern): the new nav labels use the same keys. If a route has no existing `nav.X` key, fall back to a `Title-Cased` form of the path segment.
- Inject one inline-SVG icon per route using the keyword mapping from Phase 2.
- The sidebar footer renders detected preserved components via slots:
  ```vue
  <slot name="footer">
    <!-- consumer slots ProfileMenu / LanguageSwitcher here -->
  </slot>
  ```

#### Step 3 — App.vue

Restructure the template top-level:

```vue
<template>
  <div class="app-shell">
    <Sidebar :routes="navRoutes">
      <template #footer>
        <LanguageSwitcher />
        <ProfileMenu ... />
      </template>
    </Sidebar>
    <main class="app-main">
      <FilterBar v-if="$route.meta.showFilters !== false" />
      <router-view />
    </main>
  </div>
</template>
```

Notes:
- Move `LanguageSwitcher` and `ProfileMenu` from the old top-nav header into the sidebar's `#footer` slot (preserve any props/events they had).
- `FilterBar` (if it was at App.vue level) goes inside `<main>`, above `<router-view>`.
- `navRoutes` is a const exported from the script — list of `{ path, labelKey, icon }`. Hard-code the route list at the App.vue level (not auto-derived from the router) so the labels and icons are explicit; this also lets users hide routes from nav by simply omitting them.
- Update unscoped styles: `.app-shell` becomes `display: grid; grid-template-columns: var(--sidebar-width) 1fr; min-height: 100vh;`. `.app-main` becomes `display: flex; flex-direction: column; min-width: 0;` (the `min-width: 0` prevents grid overflow).
- Preserve every existing CSS class name (`.card`, `.stat-card`, `.page-header`, etc.) — only re-target their padding/colors to use tokens.

#### Step 4 — Type & spacing repaint (App.vue unscoped styles)

Apply these targeted updates to existing global classes:
- `body { font-family: var(--font-sans); font-size: var(--text-base); line-height: 1.55; color: var(--color-ink); background: var(--color-bg); }`
- `.card { background: var(--color-surface); border: 1px solid var(--color-border); border-radius: var(--radius-lg); box-shadow: var(--shadow-sm); }`
- `.card-header { padding: var(--space-4) var(--space-5); border-bottom: 1px solid var(--color-border); }`
- `.page-header { padding: var(--space-6) var(--space-8) var(--space-4); }`
- `.page-header h2 { font-size: var(--text-xl); font-weight: 600; letter-spacing: -0.01em; }`
- `.page-header p { color: var(--color-muted); margin: var(--space-1) 0 0; }`
- `.stat-card` padding/colors: `padding: var(--space-4); border: 1px solid var(--color-border); border-radius: var(--radius-md);`

Do not redefine semantic colors (`.danger`, `.success`, `.warning`) unless they currently use harsh saturated colors; if so, soften them to the slate-compatible variants. Otherwise leave them alone.

#### Step 5 — Delegation guard

Before writing any `.vue` file, check whether `.claude/agents/vue-expert.md` exists at the project root. If so, **do not edit the .vue files directly** — instead delegate by calling the `Agent` tool:

```
Agent({
  description: "Apply sidebar redesign to App.vue",
  subagent_type: "vue-expert",
  prompt: "<full instructions for the specific .vue change, including the exact new template and styles>"
})
```

For non-`.vue` files (`tokens.css`, `main.js`), edit directly.

#### Constraints during Phase 3 (must hold)

- Do **not** touch routes, composables, API clients, or business logic.
- Do **not** rename existing CSS classes; only redefine their properties via tokens.
- Do **not** introduce new npm dependencies. Icons are inline SVG; fonts are system stack.
- Preserve i18n keys exactly; if labels were hard-coded, leave them hard-coded (don't introduce i18n).
- Preserve every existing event binding and prop on preserved components (ProfileMenu, LanguageSwitcher, FilterBar).

---

### Phase 4 — Verify

After Phase 3 writes complete:

1. **Confirm dev server.** `lsof -ti:<port>` on the Vite default (5173 or 3000 depending on `vite.config`). If not running, start it in the background: `cd <client-dir> && npm run dev > /tmp/<project>-frontend.log 2>&1 &`.

2. **Visual check.** Prefer Playwright MCP if available:
   - Navigate to the index URL.
   - `browser_snapshot` and confirm:
     - An `<aside>` element exists in the layout.
     - Its computed width matches `--sidebar-width`.
     - The active nav link for the current route has the active class.
     - The footer slot rendered the preserved components.
   - `browser_console_messages` (errors only) — confirm no new errors above the pre-change baseline.

3. **Fallback if Playwright unavailable.** `curl -s http://localhost:<port> | grep -E '(aside|sidebar)'` — should show the sidebar markup. Then `open http://localhost:<port>` so the user can eyeball it.

4. **Summary report.** Tell the user:
   - Files changed (paths only)
   - Routes detected and icon assigned to each
   - Preserved components and where they landed
   - Pre-existing console errors carried over (so they don't blame the redesign)
   - Anything skipped/punted (Vue 2 components, non-router routes, etc.)

---

## Reference files

- [`reference/tokens.css`](reference/tokens.css) — Linear-style design tokens. Copy or inline into the target project.
- [`reference/sidebar.vue.example`](reference/sidebar.vue.example) — Full Sidebar.vue template with comments.
- [`reference/icons.md`](reference/icons.md) — Inline-SVG icon library + path-keyword mapping table.

## Out of scope (v1)

- Stripe / Notion style flavors — flag is parsed but only `linear` is implemented.
- Mobile collapsible drawer — desktop-first fixed sidebar.
- Dark mode tokens — separate layer, not included.
- Pixel-diff visual regression — verification is structural + console-error only.
- Vue 2 — skill refuses to run.

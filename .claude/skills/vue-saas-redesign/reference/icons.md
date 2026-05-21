# Icon library — vue-saas-redesign

Inline SVG icons, 16x16, 1.5px stroke, `currentColor`. No external dependencies. Copy the appropriate SVG block per route into the `route.icon` field of the Sidebar's `routes` prop.

All icons share the same opening attributes:
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none"
     stroke="currentColor" stroke-width="1.5"
     stroke-linecap="round" stroke-linejoin="round">
```

## Route-keyword to icon mapping

Apply case-insensitive substring matching against each route's `path`. Longest match wins. Fall back to `Square` if nothing matches.

| Path contains | Icon |
|---|---|
| `/` (exactly `/`), `dashboard`, `overview`, `home` | Home |
| `inventory`, `stock`, `products`, `items` | Box |
| `orders`, `cart`, `checkout` | ShoppingCart |
| `restocking`, `replenish`, `reorder` | RefreshCw |
| `finance`, `spending`, `billing`, `revenue` | DollarSign |
| `demand`, `forecast`, `trend`, `growth` | TrendingUp |
| `reports`, `analytics`, `metrics`, `insights` | BarChart3 |
| `docs`, `notes`, `articles` | FileText |
| `settings`, `config`, `preferences` | Settings |
| `users`, `team`, `members`, `people` | Users |
| (anything else) | Square |

---

## Icons

### Home
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M3 9.5L12 3l9 6.5V20a1 1 0 0 1-1 1h-5v-7h-6v7H4a1 1 0 0 1-1-1V9.5z"/>
</svg>
```

### Box
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M21 8L12 3 3 8v8l9 5 9-5V8z"/>
  <path d="M3 8l9 5 9-5"/>
  <path d="M12 13v8"/>
</svg>
```

### ShoppingCart
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <circle cx="9" cy="20" r="1.5"/>
  <circle cx="18" cy="20" r="1.5"/>
  <path d="M2 3h2.5l3 12.5h12L22 7H6"/>
</svg>
```

### RefreshCw
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M21 12a9 9 0 1 1-3.5-7.1"/>
  <path d="M21 4v5h-5"/>
</svg>
```

### DollarSign
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <line x1="12" y1="2" x2="12" y2="22"/>
  <path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/>
</svg>
```

### TrendingUp
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <polyline points="3 17 9 11 13 15 21 7"/>
  <polyline points="15 7 21 7 21 13"/>
</svg>
```

### BarChart3
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <line x1="6"  y1="20" x2="6"  y2="12"/>
  <line x1="12" y1="20" x2="12" y2="4"/>
  <line x1="18" y1="20" x2="18" y2="14"/>
  <line x1="3"  y1="20" x2="21" y2="20"/>
</svg>
```

### FileText
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
  <polyline points="14 2 14 8 20 8"/>
  <line x1="8" y1="13" x2="16" y2="13"/>
  <line x1="8" y1="17" x2="16" y2="17"/>
</svg>
```

### Settings
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <circle cx="12" cy="12" r="3"/>
  <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 1 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09a1.65 1.65 0 0 0-1-1.51 1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 1 1-2.83-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09a1.65 1.65 0 0 0 1.51-1 1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 1 1 2.83-2.83l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 1 1 2.83 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/>
</svg>
```

### Users
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/>
  <circle cx="9" cy="7" r="4"/>
  <path d="M23 21v-2a4 4 0 0 0-3-3.87"/>
  <path d="M16 3.13a4 4 0 0 1 0 7.75"/>
</svg>
```

### Square (fallback)
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <rect x="4" y="4" width="16" height="16" rx="2"/>
</svg>
```

---

## Notes

- Icons inherit color from `currentColor`, so they pick up the nav item's `color` value automatically (including the active state's `--color-accent-ink`).
- Stroke width `1.5` reads well at 16x16 on retina; bump to `1.75` only if rendering at 20x20+.
- For projects that already use a Vue icon library (lucide-vue-next, heroicons), prefer those over inline SVG — but the skill does not add new dependencies. Inline SVG is the v1 default.

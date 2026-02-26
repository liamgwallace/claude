---
name: Retool Custom Components
description: "Build production-grade Retool custom components in React/TypeScript. Use when the user wants to create, design, or improve a custom component for Retool using the CCL (Custom Component Library) system. Covers component architecture, Retool TypeScript API integration, inspector properties, event handlers, data flow, styling, sizing, loading states, and deployment. Trigger on mentions of 'Retool custom component', 'Retool CCL', or requests to build React components for Retool apps."
---

# Retool Custom Components Skill

Build custom React/TypeScript components for Retool that feel native, are highly configurable, and integrate seamlessly with Retool's data model, event system, and Inspector UI.

## Quick Reference

| Task | Action |
|------|--------|
| Understand the user's needs | Follow the [Planning Phase](#planning-phase) |
| Build a new component | Follow the [Component Architecture](#component-architecture) and [Implementation Guide](#implementation-guide) |
| Add Inspector properties | See [Inspector Properties Reference](#inspector-properties-reference) |
| Wire up events | See [Event Handlers](#event-handlers) |
| Handle data from queries | See [Data Flow Patterns](#data-flow-patterns) |
| Inherit app theme colours | See [Theme Integration](#theme-integration) |
| Style the component | See [Styling & Appearance](#styling--appearance) |
| Debug issues | See [Debugging Guide](#debugging-guide) |
| Deploy | See [Deployment](#deployment) |

Read [REFERENCE.md](REFERENCE.md) for the complete TypeScript API reference, theme property list, and advanced patterns.
Read [PATTERNS.md](PATTERNS.md) for full example components demonstrating best practices.
Read [INTEGRATION.md](INTEGRATION.md) for the integration guide template.
Read [PLANNING_TEMPLATE.md](PLANNING_TEMPLATE.md) for the full requirements planning template.

**Deliverables:** When building a component for handoff to a builder, produce:
1. The component code (`.tsx` files)
2. A `queries/` folder with example SQL/JS queries for each data input
3. An `INTEGRATION.md` with step-by-step wiring instructions

---

## Planning Phase

**Before writing any code, gather requirements across these four areas.** Ask targeted questions to fill in gaps. A well-planned component avoids costly rewrites.

### 1. Data Requirements
- What data does this component consume? (SQL query results, JS transformer output, static values)
- What is the data shape? Retool queries return `[{id: 1, name: "Alice"}, ...]`. Design inputs to accept this shape.
- What data does the component output? (selected row, form values, computed results)
- Does it need to trigger queries? Does it need real-time updates?

### 2. Functionality Requirements
- What is the core interaction? (display, select, edit, navigate, visualise)
- What states must it handle? (loading, empty, error, disabled, readonly)
- Does it need filtering, sorting, or pagination?

### 3. UX/UI Requirements
- Where does this sit on the page? What size by default? (Retool grid: width 1-12 columns, height in ~8px row units)
- What should be configurable in the Inspector vs hardcoded?
- Does it need a loading spinner, empty state, or error display?

### 4. Integration Requirements
- Which event handlers should it expose?
- Which properties need to be static vs dynamic (bound to queries/JS)?
- Does it need to work inside a ListView or Form?

**Output a brief spec before proceeding to code:**

```
## Component Spec: [Name]

**Purpose:** [one sentence]
**Default Size:** [width]x[height] grid units
**Data In:** [expected input shape]
**Data Out:** [state values exposed to Retool]
**Events:** [list of event callbacks]
**Inspector Properties:** [list with types and defaults]
**States:** loading | empty | error | ready | disabled
```

---

## Component Architecture

### Project Structure

```
my-component-library/
├── package.json
├── tsconfig.json
└── src/
    ├── index.tsx              # Exports all components
    └── MyComponent/
        ├── MyComponent.tsx    # Main component logic
        ├── types.ts           # TypeScript interfaces
        └── styles.ts          # Style constants
```

**Critical:** Only components exported from `src/index.tsx` are detected by Retool.

```tsx
// src/index.tsx
export { MyComponent } from './MyComponent/MyComponent';
```

### Component Skeleton

Every component follows this structure. See [PATTERNS.md](PATTERNS.md) for a fully fleshed-out example.

```tsx
import React, { FC, useMemo, useCallback } from 'react';
import { Retool } from '@tryretool/custom-component-support';

export const MyComponent: FC = () => {
  // ── 1. Component Settings (size) ──────────────────────
  Retool.useComponentSettings({
    defaultHeight: 40,  // ~320px (rows are ~8px each)
    defaultWidth: 6,    // half the 12-column grid
  });

  // ── 2. Configuration Properties (Inspector) ──────────
  const [title] = Retool.useStateString({
    name: "title",
    initialValue: "My Component",
    label: "Title",
    description: "Display title shown in the header",
  });

  // ── 3. Data Properties (from queries/JS) ─────────────
  const [data] = Retool.useStateArray({
    name: "data",
    initialValue: [],
    label: "Data",
    description: "Array of objects from a query (e.g. {{query1.data}})",
  });

  // ── 4. Behaviour Properties ───────────────────────────
  // Use inspector: "text" (not "checkbox") so builders can bind expressions
  const [loading] = Retool.useStateBoolean({
    name: "loading",
    initialValue: false,
    label: "Loading",
    inspector: "text",
    description: "Show loading state (bind to {{query1.isFetching}})",
  });

  const [disabled] = Retool.useStateBoolean({
    name: "disabled",
    initialValue: false,
    label: "Disabled",
    inspector: "text",
    description: "Disable all interactions",
  });

  // ── 5. Theme ───────────────────────────────────────────
  // Always include — builder binds {{ theme }} to inherit app colours
  const [appTheme] = Retool.useStateObject({
    name: "appTheme",
    initialValue: {},
    label: "App Theme",
    description: "Bind to {{ theme }} to inherit app colours, fonts, and border radius",
  });

  // ── 6. Output State (hidden from Inspector) ───────────
  const [selectedItem, setSelectedItem] = Retool.useStateObject({
    name: "selectedItem",
    initialValue: {},
    inspector: "hidden",
  });

  // ── 7. Event Callbacks ────────────────────────────────
  const onItemClick = Retool.useEventCallback({ name: "itemClick" });

  // ── 8. Internal Logic ─────────────────────────────────
  const handleItemClick = useCallback((item: any, index: number) => {
    if (disabled) return;
    setSelectedItem(item);
    onItemClick();  // Fire AFTER setting state so Retool can read it
  }, [disabled, setSelectedItem, onItemClick]);

  // ── 9. Render ─────────────────────────────────────────
  const items = Array.isArray(data) ? data : [];

  if (loading) return <div>Loading...</div>;          // replace with real spinner
  if (items.length === 0) return <div>No data</div>;  // replace with real empty state

  return (
    <div style={{ width: '100%', height: '100%', overflow: 'auto' }}>
      {/* Component UI */}
    </div>
  );
};
```

---

## Inspector Properties Reference

### Choosing the Right Hook

| Data Type | Hook | Inspector Options | Use For |
|-----------|------|-------------------|---------|
| Text | `useStateString` | `"text"`, `"hidden"` | Labels, titles, placeholder text, CSS values |
| Number | `useStateNumber` | `"text"`, `"hidden"` | Counts, sizes, thresholds, indices |
| Boolean | `useStateBoolean` | `"checkbox"`, `"text"`, `"hidden"` | Feature toggles, visibility flags, loading |
| Enum | `useStateEnumeration` | `"segmented"`, `"select"`, `"hidden"` | Mode selection, variants, alignment |
| Object | `useStateObject` | `"text"`, `"hidden"` | Complex config, style overrides, query results |
| Array | `useStateArray` | `"text"`, `"hidden"` | Data arrays, column configs, option lists |

### Inspector Modes

- **`"text"`** — Text input. Builder can type literal values OR expressions like `{{ query1.data }}`. Default for most types.
- **`"checkbox"`** — Boolean only. Shows a toggle. **Cannot accept expressions** — use `"text"` when the builder needs to bind `{{ query.isFetching }}` or `{{ !currentUser.isAdmin }}`.
- **`"segmented"`** — Enum only. Segmented control. Best for 2-4 options.
- **`"select"`** — Enum only. Dropdown. Better for 5+ options.
- **`"hidden"`** — Property exists on the component (accessible via `{{ componentId.propertyName }}`) but not shown in Inspector. **Use only for OUTPUT values** the component sets programmatically.

### Property Naming Conventions

| Category | Examples | Inspector |
|----------|----------|-----------|
| **Data inputs** | `data`, `options`, `items`, `columns` | `"text"` |
| **Labels/text** | `title`, `placeholder`, `emptyText` | `"text"` |
| **Appearance** | `variant`, `size`, `alignment` | `"segmented"` or `"select"` |
| **Feature flags** | `showHeader`, `showBorder`, `sortable` | `"checkbox"` |
| **Behaviour** | `loading`, `disabled` | `"text"` (expressions) |
| **Output (hidden)** | `selectedRow`, `selectedIndex`, `value` | `"hidden"` |

---

## Event Handlers

Events let builders wire your component into Retool — triggering queries, setting variables, showing modals, etc.

```tsx
const onRowClick = Retool.useEventCallback({ name: "rowClick" });
const onSubmit = Retool.useEventCallback({ name: "submit" });
```

### Critical Pattern: Set State BEFORE Firing Events

Events cannot carry data directly. Set state first, then fire. Retool reads the updated state in the event handler.

```tsx
const handleRowClick = useCallback((row: any, index: number) => {
  setSelectedRow(row);      // 1. Set state
  setSelectedIndex(index);
  onRowClick();             // 2. Fire event — handler reads {{ component.selectedRow }}
}, [setSelectedRow, setSelectedIndex, onRowClick]);
```

### Standard Events

| Event | When to Fire |
|-------|-------------|
| `click` / `itemClick` | User clicks the primary action |
| `selectionChange` | Selected item(s) change |
| `change` / `valueChange` | Input value changes |
| `submit` | Form/action submitted |
| `error` | Component encounters an error |

---

## Data Flow Patterns

### Consuming Retool Query Data

```tsx
// Builder sets: {{ getUsers.data }}
// Arrives as: [{id: 1, name: "Alice"}, ...]
const [data] = Retool.useStateArray({
  name: "data",
  initialValue: [],
  label: "Data source",
  description: "Bind to a query result, e.g. {{ getUsers.data }}",
});
```

### Exposing Data Back to Retool

```tsx
// Other components can use {{ myComponent.selectedRow.id }}
const [selectedRow, setSelectedRow] = Retool.useStateObject({
  name: "selectedRow",
  initialValue: {},
  inspector: "hidden",
});
```

### Defensive Data Handling

Always validate — builders may provide unexpected shapes:

```tsx
const safeData = useMemo(() => {
  if (!Array.isArray(data)) return [];
  return data.filter(item => item != null);
}, [data]);
```

---

## Theme Integration

Custom components run in a sandboxed iframe and cannot access `theme` directly. Pass it in as a property:

```tsx
const [appTheme] = Retool.useStateObject({
  name: "appTheme",
  initialValue: {},
  label: "App Theme",
  description: "Bind to {{ theme }} to inherit app colours and fonts",
});

const colors = {
  primary:          (appTheme as any)?.primary         || '#3D7FFF',
  canvas:           (appTheme as any)?.canvas           || '#F6F6F6',
  surfacePrimary:   (appTheme as any)?.surfacePrimary   || '#FFFFFF',
  danger:           (appTheme as any)?.danger           || '#DC2626',
  success:          (appTheme as any)?.success          || '#059669',
};
const borderRadius = (appTheme as any)?.borderRadius || '6px';
const fontFamily   = (appTheme as any)?.defaultFont?.name
  || '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
// NOTE: mode can be null — always check === 'dark', never use || 'light'
const isDark = (appTheme as any)?.mode === 'dark';
```

See [REFERENCE.md](REFERENCE.md) for the full list of `theme` properties.

### Rules
- **Always include `appTheme`** on every component. It costs nothing and ensures theme-awareness.
- Use theme colours as defaults; allow dedicated override properties (e.g. `accentColor`) to take priority:

```tsx
const resolvedAccent = accentColor || colors.primary;
```

> **Verifying theme binding:** Retool's default blue theme is close to the fallback defaults. To confirm binding is live, temporarily set `primary` to a bright colour in **Settings → Branding**, then revert.

---

## Styling & Appearance

Components render inside a sandboxed iframe — **only inline styles and `<style>` tags work**. External CSS files do not load.

Fill the container:

```tsx
<div style={{ width: '100%', height: '100%', overflow: 'auto', boxSizing: 'border-box' }}>
```

For scrollable layouts:

```tsx
<div style={{ width: '100%', height: '100%', display: 'flex', flexDirection: 'column' }}>
  <header style={{ flexShrink: 0 }}>{/* Fixed header */}</header>
  <main style={{ flex: 1, overflow: 'auto' }}>{/* Scrollable body */}</main>
</div>
```

CSS keyframes must be injected via `<style>`:

```tsx
<style>{`@keyframes spin { to { transform: rotate(360deg); } }`}</style>
```

### Retool Visual Language

To blend with native components, use:

```ts
const RETOOL = {
  text:         '#1A1F36',
  textMuted:    '#9CA3AF',
  border:       '#E5E7EB',
  bgHover:      '#F3F4F6',
  bgSelected:   '#EBF2FF',
  fontSizeMd:   '13px',
  paddingMd:    '8px',
  borderRadius: '6px',
};
```

### Configurable Appearance

```tsx
const [rowHeight] = Retool.useStateEnumeration({
  name: "rowHeight",
  enumDefinition: ["small", "medium", "large"] as const,
  initialValue: "medium",
  label: "Row height",
  inspector: "segmented",
  enumLabels: { small: "Small (32px)", medium: "Medium (40px)", large: "Large (52px)" },
});

const [showBorder] = Retool.useStateBoolean({
  name: "showBorder",
  initialValue: true,
  label: "Show border",
  inspector: "checkbox",
});
```

---

## Sizing Guidelines

```tsx
Retool.useComponentSettings({
  defaultHeight: 40,  // Each row is ~8px
  defaultWidth: 6,    // Out of 12 columns
});
```

| Component Type | defaultWidth | defaultHeight |
|----------------|-------------|---------------|
| Small widget (stat, badge) | 3-4 | 10-15 |
| Button/action | 3-4 | 8-10 |
| Card/detail | 4-6 | 25-40 |
| Table/list | 8-12 | 40-60 |
| Chart/visualisation | 6-12 | 35-50 |
| Full-width panel | 12 | 50-80 |

---

## Implementation Guide

### First-Time Setup

**Prerequisites:** Node.js v20+, admin permissions in your Retool org.

```bash
git clone https://github.com/tryretool/custom-component-collection-template my-components
cd my-components
npm install

# Authenticate (opens browser OAuth)
npx @tryretool/custom-component-support login

# Register library in your org
npx @tryretool/custom-component-support init
# → prompts for library name and Retool host (e.g. https://mycompany.retool.com)
```

> **Important:** The CLI is `@tryretool/custom-component-support` — there is no standalone `retool-ccl` package on npm.

After `init`, `package.json` gains a required block — **commit this file**:

```json
{
  "retoolCustomComponentLibraryConfig": {
    "entryPoint": "./src/index.tsx",
    "outputPath": "./dist"
  }
}
```

If either field is missing, dev and build will fail.

```bash
# Start dev server (hot-reload into Retool)
npm run dev

# Verify build without connecting to Retool
npx @tryretool/custom-component-support dev --dry-run
```

In Retool: open the app editor → **+ components** → scroll to **Custom Components** → drag your component onto the canvas.

For all subsequent sessions, just run `npm run dev`.

### Development Checklist

Before considering a component complete:

- [ ] All data inputs handle null/undefined/empty gracefully
- [ ] Loading state shows a spinner
- [ ] Empty state shows a helpful, configurable message
- [ ] Disabled state prevents interactions and shows visual feedback
- [ ] Events fire AFTER state is set
- [ ] Component fills its container (`width: 100%`, `height: 100%`)
- [ ] Component handles overflow with scrolling
- [ ] Inspector properties have clear labels and descriptions
- [ ] Output properties use `inspector: "hidden"`
- [ ] `appTheme` property included; component uses theme colours as defaults
- [ ] No console errors in browser devtools
- [ ] Component works when resized

### Required Deliverables (handoff)

#### `queries/` folder

For each data input, provide an example query showing the expected shape:

```
queries/
├── getData.sql          # Comment header: purpose, binding, expected output shape
├── transformData.js     # JS transformer if data needs reshaping
└── README.md            # Brief explanation of each query
```

```sql
-- Query: getData
-- Purpose: Feeds the 'data' property of MyComponent
-- Bind in Inspector: {{ getData.data }}
-- Expected output shape: [{id, title, status, assignee, created_at}, ...]

SELECT id, title, status, assignee, created_at
FROM my_table
ORDER BY created_at DESC
LIMIT 100;
```

#### `INTEGRATION.md`

See [INTEGRATION.md](INTEGRATION.md) for the template. Include:
1. Prerequisites (queries, resources needed)
2. Step-by-step: drag component, create queries, bind properties
3. Property bindings table
4. Event handler wiring
5. Connections to other components
6. Theme binding reminder (`appTheme` → `{{ theme }}`)

---

## Debugging Guide

### Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Component not appearing in panel | Not exported from `src/index.tsx` | Add export and refresh Retool |
| `npx retool-ccl` not found | No such package exists | Use `npx @tryretool/custom-component-support <command>` |
| Dev server / build fails | Missing `retoolCustomComponentLibraryConfig` in `package.json` | Add `{ "entryPoint": "./src/index.tsx", "outputPath": "./dist" }` |
| Properties not showing in Inspector | Wrong `inspector` value or missing `name` | Check hook params |
| Data always empty | Builder forgot to bind `{{ query.data }}` | Add clear `description` to the property |
| Events not triggering handlers | State set after event fires | Move `setState` calls before `eventCallback()` |
| Styles not rendering | CSS not injected into iframe | Use inline styles or `<style>` tags |
| Component blank after deploy | Still using dev mode | Pin to deployed version number in Inspector |
| Theme binding looks identical to defaults | App uses Retool's default blue — fallback colours match | Temporarily set `primary` to a bright colour in Settings → Branding |
| `useStateNumber` rejects negative values | Known Retool limitation | Use `0` as initialValue, document the limitation |
| Object state merges instead of replaces | `useStateObject` uses `Object.assign()` semantics | Set to `{}` first, then set new value |

### Logging

```tsx
console.log('[MyComponent] data:', data, 'loading:', loading);
```

Open browser DevTools → Console. Custom components run in sandboxed iframes — select the correct frame context in the console dropdown.

---

## Deployment

```bash
# Verify build compiles cleanly
npx @tryretool/custom-component-support deploy --dry-run

# Deploy immutable version
npx @tryretool/custom-component-support deploy

# In Retool app: Inspector → Version → switch from 'dev' to published version number
```

### Limitations

- Not supported in **Retool Mobile**
- Library revisions must be under **10MB** (30MB in dev mode)
- Requires **Node.js v20+** and **admin permissions**
- Must be written in **React + TypeScript**
- Not included in **PDF exports**
- Run in a **sandboxed iframe** — no direct DOM access to the parent page

---

## Anti-Patterns to Avoid

1. **Don't use `inspector: "hidden"` for configurable properties** — only for OUTPUT values
2. **Don't fire events before setting state** — Retool reads state in the event handler
3. **Don't hardcode data shapes** — always handle unexpected/missing fields
4. **Don't forget loading and empty states** — first thing users see
5. **Don't use external CSS files** — use inline styles or `<style>` tags
6. **Don't assume a fixed size** — component must work at any width/height
7. **Don't use `localStorage` or `sessionStorage`** — the iframe is sandboxed
8. **Don't create properties without descriptions** — builders won't know what to bind
9. **Don't use `inspector: "checkbox"` for `loading`/`disabled`** — use `"text"` so builders can bind `{{ query.isFetching }}`

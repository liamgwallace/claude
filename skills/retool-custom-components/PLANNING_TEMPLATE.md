# Retool Custom Component — Planning Template

Use this template before writing any code. Fill in every section. Gaps here become bugs later.

---

## 1. Component Identity

| Field | Value |
|-------|-------|
| **Name** | e.g. `StatusCardList` |
| **Purpose** | One sentence: what problem does this solve? |
| **Default size** | e.g. `8 columns × 40 rows (~320px tall)` |
| **Replaces / extends** | e.g. "replaces a native Table + custom CSS hack" |

---

## 2. Data Inputs

Things the builder binds to a query result or JS transformer. These appear in the Inspector and accept `{{ query.data }}` expressions.

| Property name | Type | Required? | Expected shape | Description shown in Inspector |
|---------------|------|-----------|----------------|-------------------------------|
| `data` | Array | Yes | `[{ id, title, status }]` | "Bind to a query result" |
| `options` | Array | No | `[{ label, value }]` | "Dropdown options" |
| `config` | Object | No | `{ pageSize, sortKey }` | "Optional config overrides" |

**Query shape contract** — describe what a correct row looks like:

```json
{
  "id": 1,
  "title": "Fix login bug",
  "status": "open",
  "assignee": "Alice",
  "priority": "high",
  "created_at": "2024-01-15T10:00:00Z"
}
```

**What to do with bad data** — e.g. missing fields, null rows, non-array value:
- [ ] Show empty state
- [ ] Filter out null rows silently
- [ ] Show error state with message
- [ ] Fall back to `[]`

---

## 3. Configuration / Settings (Inspector Properties)

Static settings the builder sets once. Not expected to change at runtime.

| Property name | Type | Inspector widget | Default | Options (if enum) | Description |
|---------------|------|-----------------|---------|-------------------|-------------|
| `title` | String | `text` | `"My Component"` | — | Header text |
| `emptyText` | String | `text` | `"No data"` | — | Message when data is empty |
| `variant` | Enum | `segmented` | `"default"` | `default`, `compact`, `detailed` | Display density |
| `colorScheme` | Enum | `select` | `"blue"` | `blue`, `green`, `red`, `gray` | Accent colour scheme |
| `rowHeight` | Enum | `segmented` | `"medium"` | `small` (32px), `medium` (40px), `large` (52px) | Row height |
| `showHeader` | Boolean | `checkbox` | `true` | — | Show/hide the column header row |
| `showBorder` | Boolean | `checkbox` | `true` | — | Wrap component in a border |
| `maxRows` | Number | `text` | `50` | — | Max rows to render |
| `accentColor` | String | `text` | `""` | — | Override accent colour (hex). Empty = use app theme |

---

## 4. Behaviour Properties

Properties that control component state. Often bound to query state expressions.

| Property name | Type | Inspector widget | Default | Typical binding | Description |
|---------------|------|-----------------|---------|-----------------|-------------|
| `loading` | Boolean | `text` | `false` | `{{ getItems.isFetching }}` | Show loading spinner |
| `disabled` | Boolean | `text` | `false` | `{{ !currentUser.isAdmin }}` | Disable all interactions |
| `hidden` | Boolean | `text` | `false` | `{{ !currentUser.canView }}` | Hide the component entirely |
| `searchable` | Boolean | `checkbox` | `false` | — | Show search/filter input |
| `sortable` | Boolean | `checkbox` | `true` | — | Allow column header sort clicks |

> **Note:** Use `inspector: "text"` (not `"checkbox"`) for `loading` and `disabled` so builders can bind expressions like `{{ query.isFetching }}`.

---

## 5. Methods (Callable from Retool)

Actions other components or event handlers can call on this component, e.g. `myComponent.clearSelection()`.

> **CCL limitation:** The CCL API does not currently support exposing callable methods directly. The workaround is a **trigger property** — a boolean or string the builder sets to signal the component to do something, then the component watches with `useEffect`.

| Method intent | Workaround property | Type | How it works |
|---------------|--------------------|----|--------------|
| `clearSelection()` | `clearSelectionTrigger` | Boolean | Builder sets to `true`; component resets selection in `useEffect`, then sets back to `false` |
| `scrollToTop()` | `scrollToTopTrigger` | Boolean | Same toggle pattern |
| `setFilter(value)` | `filterValue` | String | Builder sets this string; component filters on change |
| `refresh()` | n/a | — | Wire to the underlying query's `trigger()` method instead |

---

## 6. Theme & Appearance

| Aspect | Approach |
|--------|----------|
| **App theme** | Include `appTheme` Object property; builder binds `{{ theme }}` |
| **Colours** | Derived from `appTheme.primary`, `surfacePrimary`, etc. with hardcoded fallbacks |
| **Dark mode** | Check `appTheme.mode === 'dark'` (can be `null` — never use `\|\| 'light'`) |
| **Font** | Use `appTheme.defaultFont.name` with system-font fallback |
| **Border radius** | Use `appTheme.borderRadius` with `'6px'` fallback |
| **Custom overrides** | Allow `accentColor` string to override `primary` |

---

## 7. UX / UI Design Intent

### Layout

Describe the visual structure:

```
┌─────────────────────────────────────┐
│  [Title]              [Search box]  │  ← header (fixed)
├─────────────────────────────────────┤
│  Col A    │  Col B    │  Col C      │  ← column headers (fixed)
├─────────────────────────────────────┤
│  row 1                              │  ↑
│  row 2                              │  scrollable body
│  row 3                              │  ↓
└─────────────────────────────────────┘
```

### States

| State | What the user sees |
|-------|-------------------|
| **Loading** | Spinner centred in the component |
| **Empty** | Icon + configurable `emptyText` message |
| **Error** | Red warning icon + error message string |
| **Ready** | Normal data display |
| **Disabled** | Greyed out, cursor `not-allowed`, no click response |
| **Selected row** | Row highlighted with `primary` colour background |

### Interactions

| User action | What happens |
|-------------|-------------|
| Click a row | Row highlights; `selectedRow` state updates; `rowClick` event fires |
| Click column header | Sort by that column (toggle asc/desc); `sortColumn`/`sortDirection` update |
| Type in search box | Filters visible rows in real time; `filterText` state updates |
| Hover a row | Row background lightens (hover style) |
| Click outside (if modal-like) | Close / deselect |

### Accessibility

- [ ] Keyboard navigation (arrow keys to move selection)
- [ ] Focus ring visible
- [ ] `aria-label` on interactive elements
- [ ] Sufficient colour contrast (4.5:1 minimum)

---

## 8. Data Processing

Describe how raw input data is transformed before rendering.

| Step | Description |
|------|-------------|
| **Validate** | Check `Array.isArray(data)`, filter null rows |
| **Normalise** | Map raw field names to internal shape if needed |
| **Auto-columns** | If no `columns` prop, derive from first row's keys |
| **Filter** | Apply `filterText` to all string fields (case-insensitive) |
| **Sort** | Apply `sortColumn` + `sortDirection` |
| **Paginate** | Slice to `maxRows` |
| **Render** | Map processed rows to JSX |

All transforms should use `useMemo` to avoid recomputing on every render.

---

## 9. Outputs — Component State (readable by other components)

Properties exposed on the component that other parts of the Retool app can read, e.g. `{{ myComponent.selectedRow.id }}`.

Use `inspector: "hidden"` for all of these.

| Property name | Type | When it updates | Example value |
|---------------|------|-----------------|---------------|
| `selectedRow` | Object | When user clicks a row | `{ id: 3, title: "Fix bug", status: "open" }` |
| `selectedIndex` | Number | When user clicks a row | `2` |
| `selectedRows` | Array | Multi-select: when selection changes | `[{ id: 1 }, { id: 3 }]` |
| `filterText` | String | When user types in search | `"alice"` |
| `sortColumn` | String | When user clicks column header | `"created_at"` |
| `sortDirection` | String | When user clicks column header | `"desc"` |
| `rowCount` | Number | When data changes | `42` |
| `hasSelection` | Boolean | When selection changes | `true` |
| `isValid` | Boolean | Form-like components | `true` |
| `value` | Any | Input-like components | `"typed value"` |

---

## 10. Outputs — Events

Callbacks the builder wires to actions (trigger queries, show modals, set variables, navigate, etc.).

| Event name | When it fires | State to read in handler |
|------------|--------------|--------------------------|
| `rowClick` | User clicks a row | `{{ myComponent.selectedRow }}` |
| `selectionChange` | Selected rows change | `{{ myComponent.selectedRows }}` |
| `filterChange` | Search text changes | `{{ myComponent.filterText }}` |
| `sortChange` | Column sort changes | `{{ myComponent.sortColumn }}`, `{{ myComponent.sortDirection }}` |
| `submit` | User submits a form | `{{ myComponent.value }}` |
| `change` | Input value changes | `{{ myComponent.value }}` |
| `error` | Component hits an error | `{{ myComponent.errorMessage }}` |
| `load` | Component finishes loading | — |

> **Rule:** Always set state **before** firing the event. The handler reads state synchronously.

---

## 11. Integration Requirements

| Question | Answer |
|----------|--------|
| Which Retool queries feed this component? | e.g. `getWorkOrders` (SQL), `transformOrders` (JS) |
| Which components does it interact with? | e.g. filters a `Table1`, opens `Modal1` on row click |
| Does it live inside a ListView or Form? | Yes / No |
| Does it need to trigger queries? | e.g. `updateStatus.trigger()` on row click |
| Does it need `currentUser` or `urlparams`? | Yes / No |

---

## 12. Deliverables Checklist

- [ ] `ComponentName.tsx` — main component
- [ ] `types.ts` — TypeScript interfaces for data shape
- [ ] `queries/getData.sql` — example query for each data input
- [ ] `queries/transformData.js` — JS transformer if data needs reshaping
- [ ] `queries/README.md` — explains each query
- [ ] `INTEGRATION.md` — step-by-step wiring guide for the builder
- [ ] Export added to `src/index.tsx`

---

## Example: `StatusCardList`

A quick filled-in example to show how this template looks in practice.

### Identity
- **Name:** `StatusCardList`
- **Purpose:** Displays work orders as cards grouped by status with click-to-select
- **Default size:** `8 × 45`

### Data Inputs
- `data: Array` — `[{ id, title, status, assignee, priority }]` — bind to `{{ getWorkOrders.data }}`

### Settings
- `title: String` — `"Work Orders"` — header label
- `groupBy: Enum (segmented)` — `status | priority | assignee` — grouping field
- `showAssignee: Boolean (checkbox)` — `true` — show assignee chip on card

### Behaviour
- `loading: Boolean (text)` — bind to `{{ getWorkOrders.isFetching }}`
- `disabled: Boolean (text)` — bind to `{{ !currentUser.isAdmin }}`

### UX
- Cards grouped under collapsible section headers
- Selected card gets a blue left border
- Empty group shows a faint "No items" label
- Loading shows skeleton cards

### Data Processing
1. Validate array, filter nulls
2. Group rows by `groupBy` field using `useMemo`
3. Sort groups alphabetically; sort cards within group by `priority`
4. Render groups → cards

### Outputs (state)
- `selectedCard: Object (hidden)` — the clicked card object
- `selectedId: Number (hidden)` — `selectedCard.id`
- `selectedGroup: String (hidden)` — the group the card belongs to

### Outputs (events)
- `cardClick` — fires after `selectedCard` is set; handler can run `updateStatus.trigger()`
- `groupToggle` — fires when a group is collapsed/expanded

### Queries needed
```sql
-- queries/getWorkOrders.sql
SELECT id, title, status, assignee, priority, created_at
FROM work_orders
WHERE status != 'archived'
ORDER BY created_at DESC;
```

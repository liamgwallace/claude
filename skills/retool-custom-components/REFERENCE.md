# Retool Custom Component — TypeScript API Reference

Complete reference for the `@tryretool/custom-component-support` package.

---

## State Hooks

All state hooks follow the same pattern: they create a property on the component that can be read and written from both the component code and the Retool Inspector.

### useStateString

```tsx
const [value, setValue] = Retool.useStateString({
  name: "myString",        // Required. Alphanumeric, no spaces.
  initialValue: "default", // Optional. Value when first dragged to canvas.
  label: "My String",      // Optional. Inspector label override.
  description: "Tooltip",  // Optional. Inspector tooltip.
  inspector: "text",       // "text" (default) | "hidden"
});
```

**Returns:** `readonly [string, (newValue: string) => void]`

### useStateNumber

```tsx
const [value, setValue] = Retool.useStateNumber({
  name: "myNumber",
  initialValue: 0,         // NOTE: Negative literals may error. Use 0 and document.
  label: "My Number",
  description: "Tooltip",
  inspector: "text",       // "text" (default) | "hidden"
});
```

**Returns:** `readonly [number, (newValue: number) => void]`

**Known issue:** `initialValue` with negative numbers (e.g. `-119.75`) causes a build error. Use `0` as default and document that the builder should set the value in the Inspector.

### useStateBoolean

```tsx
const [value, setValue] = Retool.useStateBoolean({
  name: "myBool",
  initialValue: false,
  label: "My Toggle",
  description: "Tooltip",
  inspector: "checkbox",   // "checkbox" | "text" | "hidden"
});
```

**Returns:** `readonly [boolean, (newValue: boolean) => void]`

**Tip:** Use `inspector: "text"` (not `"checkbox"`) when the builder needs to bind expressions like `{{ query.isFetching }}` or `{{ !currentUser.isAdmin }}`. Checkbox only accepts literal true/false.

### useStateEnumeration

```tsx
const [value, setValue] = Retool.useStateEnumeration({
  name: "myEnum",
  enumDefinition: ["optionA", "optionB", "optionC"] as const,
  initialValue: "optionA",
  enumLabels: {             // Optional. Display names for each option.
    optionA: "Option A",
    optionB: "Option B",
    optionC: "Option C",
  },
  label: "My Enum",
  description: "Tooltip",
  inspector: "segmented",  // "segmented" | "select" | "hidden"
});
```

**Returns:** `readonly [T[number], (newValue: T[number]) => void]`

- Use `"segmented"` for 2-4 options (horizontal button group)
- Use `"select"` for 5+ options (dropdown)
- `enumDefinition` values must be alphanumeric with no spaces

### useStateObject

```tsx
const [value, setValue] = Retool.useStateObject({
  name: "myObject",
  initialValue: { key: "default" },
  label: "Config",
  description: "Tooltip",
  inspector: "text",       // "text" | "hidden"
});
```

**Returns:** `readonly [SerializableObject, (updates: SerializableObject) => void]`

**IMPORTANT:** The setter uses `Object.assign()` semantics — it merges the update into existing state rather than replacing it. To fully replace:
```tsx
// Reset then set
setValue({});                    // Clear existing
setValue({ newKey: "newValue" }); // Set new — but this merges into {}
```

### useStateArray

```tsx
const [value, setValue] = Retool.useStateArray({
  name: "myArray",
  initialValue: [],
  label: "Items",
  description: "Tooltip",
  inspector: "text",       // "text" | "hidden"
});
```

**Returns:** `readonly [SerializableArray, (newValue: SerializableArray) => void]`

---

## Event Callbacks

### useEventCallback

```tsx
const onMyEvent = Retool.useEventCallback({ name: "myEvent" });

// Usage:
<button onClick={() => {
  setSelectedItem(item);  // Set state FIRST
  onMyEvent();            // Then fire event
}}>Click</button>
```

**Returns:** `() => void`

Events **cannot carry payload data**. The pattern is:
1. Set hidden state properties with the data you want to expose
2. Fire the event callback
3. In Retool's event handler, reference `{{ componentId.propertyName }}`

---

## Component Settings

### useComponentSettings

```tsx
Retool.useComponentSettings({
  defaultWidth: 6,     // Columns (1-12). Optional.
  defaultHeight: 40,   // Rows (~8px each). Optional.
});
```

Sets the initial size when the component is first dragged onto the canvas. Users can still resize freely after placement.

---

## Retool Theme Object Reference

The `theme` object is accessible anywhere in a Retool app and contains all app-level styling tokens. Pass it into custom components via a `useStateObject` property bound to `{{ theme }}`.

### All Theme Properties

```typescript
interface RetoolTheme {
  // Core colours
  primary: string;           // "#EB5A2A" — default accent colour
  secondary: string;         // "#EB5A2A" — optional secondary (falls back to primary)
  tertiary: string;          // null — optional tertiary (falls back to secondary)
  canvas: string;            // "#F6F6F6" — app background
  surfacePrimary: string;    // "#FFFFFF" — container/table background
  surfaceSecondary: string;  // "#FFFFFF" — input background

  // Status colours
  danger: string;            // "#DC2626" — errors, destructive actions
  info: string;              // "#3170F9" — neutral information
  warning: string;           // "#CD6F00" — warnings
  success: string;           // "#059669" — success, positive trends
  highlight: string;         // search match highlighting colour

  // Typography
  defaultFont: {
    name: string;            // "Sohne, -apple-system" — font family
    source: unknown;
  };
  h1Font: { fontWeight: string; fontSize: string };  // e.g. "700", "36px"
  h2Font: { fontWeight: string; fontSize: string };
  h3Font: { fontWeight: string; fontSize: string };
  h4Font: { fontWeight: string; fontSize: string };
  h5Font: { fontWeight: string; fontSize: string };
  h6Font: { fontWeight: string; fontSize: string };
  labelFont: { fontWeight: string; name: string; source: unknown };
  labelEmphasizedFont: { fontWeight: string; name: string; source: unknown };

  // Metrics
  borderRadius: string;      // "4px" — default border radius

  // Elevation (box-shadow CSS values)
  lowElevation: string;
  mediumElevation: string;
  highElevation: string;

  // Mode — can be null; always check === 'dark', never use || 'light'
  mode: 'light' | 'dark' | null;

  // Auto-assignment colours (for chart series etc.)
  automatic: string[];       // Array of hex colours
}
```

### Accessing in Retool Expressions

```
{{ theme.primary }}              → "#EB5A2A"
{{ theme.canvas }}               → "#F6F6F6"
{{ theme.defaultFont.name }}     → "Sohne, -apple-system"
{{ theme.h1Font.fontSize }}      → "36px"
{{ theme.borderRadius }}         → "4px"
{{ theme.mode }}                 → "light"
{{ theme.danger }}               → "#DC2626"
```

### Programmatic Mode Switching

```javascript
// In a Retool JS query or event handler:
theme.setMode('dark', { persist: true });
```

---

## Advanced Patterns

### Column Configuration Pattern

For table-like components, accept column definitions as a structured array:

```tsx
const [columns] = Retool.useStateArray({
  name: "columns",
  initialValue: [],
  label: "Columns",
  description: 'Array of column configs. E.g. {{ [{key:"name", label:"Name", width:200}, {key:"email", label:"Email"}] }}',
});

// With auto-detection fallback
const resolvedColumns = useMemo(() => {
  if (Array.isArray(columns) && columns.length > 0) {
    return columns as Array<{ key: string; label?: string; width?: number; sortable?: boolean }>;
  }
  // Auto-detect from data
  if (safeData.length > 0) {
    return Object.keys(safeData[0]).map(key => ({
      key,
      label: key.replace(/([A-Z])/g, ' $1').replace(/^./, s => s.toUpperCase()),
    }));
  }
  return [];
}, [columns, safeData]);
```

### Pagination Pattern

```tsx
const [pageSize] = Retool.useStateNumber({
  name: "pageSize",
  initialValue: 10,
  label: "Page size",
  description: "Number of rows per page. Set 0 for no pagination.",
});

const [currentPage, setCurrentPage] = Retool.useStateNumber({
  name: "currentPage",
  initialValue: 1,
  inspector: "hidden",
});

const onPageChange = Retool.useEventCallback({ name: "pageChange" });

const totalPages = useMemo(() => {
  if (pageSize <= 0) return 1;
  return Math.ceil(safeData.length / pageSize);
}, [safeData.length, pageSize]);

const pagedData = useMemo(() => {
  if (pageSize <= 0) return safeData;
  const start = (currentPage - 1) * pageSize;
  return safeData.slice(start, start + pageSize);
}, [safeData, currentPage, pageSize]);
```

### Search/Filter Pattern

```tsx
const [searchable] = Retool.useStateBoolean({
  name: "searchable",
  initialValue: false,
  label: "Enable search",
  inspector: "checkbox",
});

const [searchTerm, setSearchTerm] = Retool.useStateString({
  name: "searchTerm",
  initialValue: "",
  inspector: "hidden",
});

const filteredData = useMemo(() => {
  if (!searchable || !searchTerm.trim()) return safeData;
  const term = searchTerm.toLowerCase();
  return safeData.filter(item =>
    Object.values(item).some(val =>
      String(val).toLowerCase().includes(term)
    )
  );
}, [safeData, searchable, searchTerm]);
```

### Multi-Selection Pattern

```tsx
const [selectionMode] = Retool.useStateEnumeration({
  name: "selectionMode",
  enumDefinition: ["single", "multi", "none"] as const,
  initialValue: "single",
  label: "Selection mode",
  inspector: "segmented",
});

const [selectedRows, setSelectedRows] = Retool.useStateArray({
  name: "selectedRows",
  initialValue: [],
  inspector: "hidden",
});

const [selectedIndices, setSelectedIndices] = Retool.useStateArray({
  name: "selectedIndices",
  initialValue: [],
  inspector: "hidden",
});

const onSelectionChange = Retool.useEventCallback({ name: "selectionChange" });

const handleRowClick = useCallback((row: any, index: number) => {
  if (selectionMode === "none") return;
  
  if (selectionMode === "single") {
    setSelectedRows([row]);
    setSelectedIndices([index]);
  } else {
    // Toggle selection
    const currentIndices = selectedIndices as number[];
    if (currentIndices.includes(index)) {
      setSelectedRows((selectedRows as any[]).filter((_, i) => currentIndices[i] !== index));
      setSelectedIndices(currentIndices.filter(i => i !== index));
    } else {
      setSelectedRows([...(selectedRows as any[]), row]);
      setSelectedIndices([...currentIndices, index]);
    }
  }
  onSelectionChange();
}, [selectionMode, selectedRows, selectedIndices]);
```

### Keyboard Accessibility Pattern

```tsx
const handleKeyDown = useCallback((e: React.KeyboardEvent, row: any, index: number) => {
  switch (e.key) {
    case 'Enter':
    case ' ':
      e.preventDefault();
      handleRowClick(row, index);
      break;
    case 'ArrowDown':
      e.preventDefault();
      // Focus next row
      break;
    case 'ArrowUp':
      e.preventDefault();
      // Focus previous row
      break;
  }
}, [handleRowClick]);
```

### Tooltip / Hover Detail Pattern

```tsx
const [hoveredItem, setHoveredItem] = Retool.useStateObject({
  name: "hoveredItem",
  initialValue: {},
  inspector: "hidden",
});

const onHover = Retool.useEventCallback({ name: "hover" });

// On mouse enter:
const handleMouseEnter = (item: any) => {
  setHoveredItem(item);
  onHover();
};
```

---

## NPM Packages That Work Well in Retool Components

These packages can be `npm install`ed and used in your component code:

| Package | Use Case |
|---------|----------|
| `recharts` | Charts and data visualisation |
| `d3` | Advanced data visualisation |
| `plotly.js` | Interactive charts |
| `date-fns` / `dayjs` | Date formatting and parsing |
| `lodash` | Utility functions (sorting, grouping, debouncing) |
| `react-window` / `react-virtualized` | Virtualised lists for large datasets |
| `react-dnd` | Drag and drop |
| `react-spring` / `framer-motion` | Animations |
| `ag-grid-community` | Advanced data grids |
| `highlight.js` / `prismjs` | Code syntax highlighting |
| `marked` / `react-markdown` | Markdown rendering |
| `fuse.js` | Fuzzy search |

Install and use like any React project:
```bash
npm install recharts
```

```tsx
import { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer } from 'recharts';
```

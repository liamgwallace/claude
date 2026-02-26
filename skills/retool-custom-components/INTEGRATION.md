# Integration Guide Template

**When building a component, generate an INTEGRATION.md specific to that component following this template structure.**

---

# [Component Name] — Integration Guide

## Prerequisites

- Retool app with the custom component library deployed (version X.X or later)
- Database resource configured (e.g. PostgreSQL, MySQL, or Retool Database)
- [List any other requirements]

## Step 1: Create Queries

Create the following queries in your Retool app. The SQL files are in the `queries/` folder for reference.

### Query: `getData`

- **Type:** SQL (your_database_resource)
- **SQL:** See `queries/getData.sql`
- **Trigger:** Run on page load
- **Purpose:** Provides the main data for the component

### Query: `getOptions` (if applicable)

- **Type:** SQL or JS Transformer
- **SQL:** See `queries/getOptions.sql`
- **Purpose:** Provides dropdown/filter options

### Transformer: `transformData` (if applicable)

- **Type:** JavaScript Transformer
- **Code:** See `queries/transformData.js`
- **Purpose:** Reshapes query data into the format the component expects

## Step 2: Add the Component

1. Open the **Add components** panel (+ icon)
2. Find **[Component Name]** under your custom component library section
3. Drag it onto the canvas
4. Resize to approximately **[width] × [height]** grid units

## Step 3: Bind Properties

Configure these properties in the **Inspector** panel (right side):

| Property | Value | Notes |
|----------|-------|-------|
| **Data** | `{{ getData.data }}` | Main data source |
| **Loading** | `{{ getData.isFetching }}` | Shows spinner while query runs |
| **Disabled** | `false` | Set to `{{ !currentUser.isAdmin }}` to restrict |
| **App Theme** | `{{ theme }}` | Inherits app colours and fonts |
| **Empty state text** | `"No records found"` | Customise as needed |
| **[Other properties]** | [values] | [notes] |

## Step 4: Wire Event Handlers

Add event handlers for each event the component exposes:

### On `itemClick`

| Setting | Value |
|---------|-------|
| **Action** | Run query |
| **Query** | `getItemDetail` |
| **Additional scope** | `{ id: {{ ComponentName1.selectedItem.id }} }` |

### On `selectionChange` (if applicable)

| Setting | Value |
|---------|-------|
| **Action** | Control component |
| **Component** | `detailPanel` |
| **Method** | Show / set value |

## Step 5: Connect to Other Components

### Filtering a Table from this component:
- On the Table's data source, add a filter: `WHERE id = {{ ComponentName1.selectedItem.id }}`
- Or use a JS transformer that filters based on the component's output

### Opening a Modal:
- Add an event handler on `itemClick` → Open Modal `editModal`
- Inside the modal, reference `{{ ComponentName1.selectedItem }}` for form defaults

### Inside a ListView:
- If placing this component inside a ListView, the `data` property should use `{{ currentItem }}` or `{{ item }}`

## Step 6: Theme Binding

**Always set the `App Theme` property to `{{ theme }}`** so the component inherits your app's colour scheme. If you change your app theme later, the component will automatically update.

To override a specific colour, use the dedicated colour properties (e.g. `Accent color`) which take priority over the theme.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Component shows "No data" | Check the query ran successfully and `data` property is bound |
| Loading spinner never stops | Verify `Loading` is bound to `{{ query.isFetching }}` not hardcoded to `true` |
| Clicks don't trigger anything | Check event handlers are added in the Inspector's Event handlers section |
| Colours don't match app | Set `App Theme` to `{{ theme }}` |
| Component is too small | Resize on canvas, or adjust `defaultHeight`/`defaultWidth` in code |

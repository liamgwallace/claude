# Retool Custom Component — Example Patterns

Complete, copy-paste-ready example components demonstrating best practices.

---

## Example 1: Data Card List

A list of interactive cards with selection, search, loading, and empty states. Good template for any list/card component.

```tsx
import React, { FC, useMemo, useCallback, useState } from 'react';
import { Retool } from '@tryretool/custom-component-support';

export const DataCardList: FC = () => {
  // ── Settings ──
  Retool.useComponentSettings({ defaultHeight: 50, defaultWidth: 8 });

  // ── Configuration (Inspector) ──
  const [title] = Retool.useStateString({
    name: "title", initialValue: "Items", label: "Title",
    description: "Header title for the card list",
  });
  const [emptyText] = Retool.useStateString({
    name: "emptyText", initialValue: "No items found", label: "Empty state text",
    description: "Message when data is empty",
  });
  const [labelField] = Retool.useStateString({
    name: "labelField", initialValue: "name", label: "Label field",
    description: "Object key to display as card title",
  });
  const [descriptionField] = Retool.useStateString({
    name: "descriptionField", initialValue: "description", label: "Description field",
    description: "Object key for card subtitle/description",
  });

  // ── Data ──
  const [data] = Retool.useStateArray({
    name: "data", initialValue: [], label: "Data",
    description: "Array of objects, e.g. {{ getItems.data }}",
  });

  // ── Appearance ──
  const [variant] = Retool.useStateEnumeration({
    name: "variant",
    enumDefinition: ["default", "compact"] as const,
    initialValue: "default", label: "Variant", inspector: "segmented",
  });
  const [showBorder] = Retool.useStateBoolean({
    name: "showBorder", initialValue: true, label: "Show border", inspector: "checkbox",
  });
  const [accentColor] = Retool.useStateString({
    name: "accentColor", initialValue: "#3D7FFF", label: "Accent color",
    description: "Hex color for selected state highlight",
  });

  // ── Behaviour ──
  const [loading] = Retool.useStateBoolean({
    name: "loading", initialValue: false, label: "Loading", inspector: "text",
    description: "Bind to {{ query.isFetching }}",
  });
  const [disabled] = Retool.useStateBoolean({
    name: "disabled", initialValue: false, label: "Disabled", inspector: "text",
  });
  const [searchable] = Retool.useStateBoolean({
    name: "searchable", initialValue: true, label: "Enable search", inspector: "checkbox",
  });

  // ── Output (hidden) ──
  const [selectedItem, setSelectedItem] = Retool.useStateObject({
    name: "selectedItem", initialValue: {}, inspector: "hidden",
  });
  const [selectedIndex, setSelectedIndex] = Retool.useStateNumber({
    name: "selectedIndex", initialValue: 0, inspector: "hidden",
  });
  const [searchTerm, setSearchTerm] = Retool.useStateString({
    name: "searchTerm", initialValue: "", inspector: "hidden",
  });

  // ── Events ──
  const onItemClick = Retool.useEventCallback({ name: "itemClick" });

  // ── Internal state ──
  const [localSearch, setLocalSearch] = useState("");

  // ── Data processing ──
  const safeData = useMemo(() => {
    if (!Array.isArray(data)) return [];
    return data.filter(item => item != null);
  }, [data]);

  const filteredData = useMemo(() => {
    if (!searchable || !localSearch.trim()) return safeData;
    const term = localSearch.toLowerCase();
    return safeData.filter(item =>
      Object.values(item).some(val => String(val ?? "").toLowerCase().includes(term))
    );
  }, [safeData, searchable, localSearch]);

  // ── Handlers ──
  const handleItemClick = useCallback((item: any, index: number) => {
    if (disabled) return;
    setSelectedItem(item);
    setSelectedIndex(index);
    onItemClick();
  }, [disabled, setSelectedItem, setSelectedIndex, onItemClick]);

  const handleSearchChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setLocalSearch(e.target.value);
    setSearchTerm(e.target.value);
  }, [setSearchTerm]);

  // ── Styles ──
  const isCompact = variant === "compact";
  const padding = isCompact ? "6px 10px" : "10px 14px";
  const fontSize = isCompact ? "12px" : "13px";

  // ── Render ──
  return (
    <div style={{
      width: '100%', height: '100%', display: 'flex', flexDirection: 'column',
      fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
      border: showBorder ? '1px solid #E5E7EB' : 'none',
      borderRadius: '6px', overflow: 'hidden', backgroundColor: '#fff',
    }}>
      <style>{`@keyframes spin { to { transform: rotate(360deg); } }`}</style>

      {/* Header */}
      <div style={{
        padding: '10px 14px', borderBottom: '1px solid #E5E7EB',
        display: 'flex', alignItems: 'center', justifyContent: 'space-between',
        flexShrink: 0,
      }}>
        <span style={{ fontWeight: 600, fontSize: '14px', color: '#1A1F36' }}>{title}</span>
        <span style={{ fontSize: '12px', color: '#9CA3AF' }}>
          {filteredData.length} item{filteredData.length !== 1 ? 's' : ''}
        </span>
      </div>

      {/* Search */}
      {searchable && (
        <div style={{ padding: '8px 14px', borderBottom: '1px solid #F3F4F6', flexShrink: 0 }}>
          <input
            type="text"
            placeholder="Search..."
            value={localSearch}
            onChange={handleSearchChange}
            disabled={disabled}
            style={{
              width: '100%', padding: '6px 10px', fontSize: '13px',
              border: '1px solid #E5E7EB', borderRadius: '4px', outline: 'none',
              boxSizing: 'border-box', backgroundColor: disabled ? '#F9FAFB' : '#fff',
            }}
          />
        </div>
      )}

      {/* Body */}
      <div style={{ flex: 1, overflow: 'auto' }}>
        {loading ? (
          <div style={{
            display: 'flex', alignItems: 'center', justifyContent: 'center',
            height: '100%', color: '#6B7280', gap: '8px',
          }}>
            <div style={{
              width: 16, height: 16, border: '2px solid #E5E7EB',
              borderTopColor: accentColor, borderRadius: '50%',
              animation: 'spin 0.6s linear infinite',
            }} />
            <span style={{ fontSize: '13px' }}>Loading...</span>
          </div>
        ) : filteredData.length === 0 ? (
          <div style={{
            display: 'flex', alignItems: 'center', justifyContent: 'center',
            height: '100%', color: '#9CA3AF', fontSize: '13px',
          }}>
            {emptyText}
          </div>
        ) : (
          filteredData.map((item, i) => {
            const isSelected = selectedIndex === i;
            return (
              <div
                key={i}
                onClick={() => handleItemClick(item, i)}
                role="button"
                tabIndex={disabled ? -1 : 0}
                onKeyDown={(e) => { if (e.key === 'Enter') handleItemClick(item, i); }}
                style={{
                  padding,
                  borderBottom: '1px solid #F3F4F6',
                  cursor: disabled ? 'default' : 'pointer',
                  backgroundColor: isSelected ? `${accentColor}10` : 'transparent',
                  borderLeft: isSelected ? `3px solid ${accentColor}` : '3px solid transparent',
                  opacity: disabled ? 0.5 : 1,
                  transition: 'background-color 0.15s, border-color 0.15s',
                }}
              >
                <div style={{ fontWeight: 500, fontSize, color: '#1A1F36' }}>
                  {item[labelField] ?? `Item ${i + 1}`}
                </div>
                {descriptionField && item[descriptionField] && (
                  <div style={{
                    fontSize: isCompact ? '11px' : '12px', color: '#6B7280',
                    marginTop: '2px',
                  }}>
                    {item[descriptionField]}
                  </div>
                )}
              </div>
            );
          })
        )}
      </div>
    </div>
  );
};
```

### Usage in Retool:
- **Data:** `{{ getItems.data }}`
- **Loading:** `{{ getItems.isFetching }}`
- **Event handler on `itemClick`:** Run query `getItemDetail` with `{{ DataCardList1.selectedItem.id }}`

---

## Example 2: Status Badge / KPI Widget

A compact stat display widget for dashboards.

```tsx
import React, { FC, useMemo } from 'react';
import { Retool } from '@tryretool/custom-component-support';

export const KPIWidget: FC = () => {
  Retool.useComponentSettings({ defaultHeight: 12, defaultWidth: 3 });

  const [label] = Retool.useStateString({
    name: "label", initialValue: "Total", label: "Label",
  });
  const [value] = Retool.useStateString({
    name: "value", initialValue: "0", label: "Value",
    description: "Main display value. Bind to {{ query.data[0].count }} or similar.",
  });
  const [subtitle] = Retool.useStateString({
    name: "subtitle", initialValue: "", label: "Subtitle",
    description: "Optional secondary text below the value",
  });
  const [color] = Retool.useStateEnumeration({
    name: "color",
    enumDefinition: ["blue", "green", "red", "yellow", "gray"] as const,
    initialValue: "blue", label: "Color", inspector: "segmented",
  });
  const [loading] = Retool.useStateBoolean({
    name: "loading", initialValue: false, label: "Loading", inspector: "text",
  });

  const onClick = Retool.useEventCallback({ name: "click" });

  const colorMap: Record<string, { bg: string; text: string; accent: string }> = {
    blue:   { bg: '#EBF2FF', text: '#1E40AF', accent: '#3D7FFF' },
    green:  { bg: '#ECFDF5', text: '#065F46', accent: '#10B981' },
    red:    { bg: '#FEF2F2', text: '#991B1B', accent: '#EF4444' },
    yellow: { bg: '#FFFBEB', text: '#92400E', accent: '#F59E0B' },
    gray:   { bg: '#F9FAFB', text: '#374151', accent: '#6B7280' },
  };
  const theme = colorMap[color] || colorMap.blue;

  return (
    <div
      onClick={onClick}
      style={{
        width: '100%', height: '100%', padding: '12px 16px',
        backgroundColor: theme.bg, borderRadius: '8px',
        fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
        display: 'flex', flexDirection: 'column', justifyContent: 'center',
        cursor: 'pointer', boxSizing: 'border-box',
        borderLeft: `4px solid ${theme.accent}`,
      }}
    >
      <style>{`@keyframes spin { to { transform: rotate(360deg); } }`}</style>
      <div style={{ fontSize: '11px', fontWeight: 600, color: theme.text, opacity: 0.7,
        textTransform: 'uppercase', letterSpacing: '0.05em' }}>
        {label}
      </div>
      {loading ? (
        <div style={{
          width: 20, height: 20, border: `2px solid ${theme.accent}33`,
          borderTopColor: theme.accent, borderRadius: '50%',
          animation: 'spin 0.6s linear infinite', marginTop: '4px',
        }} />
      ) : (
        <div style={{ fontSize: '28px', fontWeight: 700, color: theme.text, marginTop: '2px',
          lineHeight: 1.1 }}>
          {value}
        </div>
      )}
      {subtitle && (
        <div style={{ fontSize: '12px', color: theme.text, opacity: 0.6, marginTop: '4px' }}>
          {subtitle}
        </div>
      )}
    </div>
  );
};
```

---

## Example 3: Action Button Group

A set of action buttons with icons, loading states, and confirmations — like a toolbar.

```tsx
import React, { FC, useMemo, useCallback } from 'react';
import { Retool } from '@tryretool/custom-component-support';

export const ActionButtonGroup: FC = () => {
  Retool.useComponentSettings({ defaultHeight: 8, defaultWidth: 6 });

  const [buttons] = Retool.useStateArray({
    name: "buttons",
    initialValue: [
      { label: "Approve", color: "#10B981", action: "approve" },
      { label: "Reject",  color: "#EF4444", action: "reject" },
      { label: "Skip",    color: "#6B7280", action: "skip" },
    ],
    label: "Buttons",
    description: 'Array of {label, color, action} objects',
  });

  const [loading] = Retool.useStateBoolean({
    name: "loading", initialValue: false, label: "Loading", inspector: "text",
  });
  const [disabled] = Retool.useStateBoolean({
    name: "disabled", initialValue: false, label: "Disabled", inspector: "text",
  });

  const [alignment] = Retool.useStateEnumeration({
    name: "alignment",
    enumDefinition: ["left", "center", "right", "stretch"] as const,
    initialValue: "right", label: "Alignment", inspector: "segmented",
  });

  // Output
  const [clickedAction, setClickedAction] = Retool.useStateString({
    name: "clickedAction", initialValue: "", inspector: "hidden",
  });

  const onButtonClick = Retool.useEventCallback({ name: "buttonClick" });

  const safeButtons = useMemo(() => {
    if (!Array.isArray(buttons)) return [];
    return buttons as Array<{ label: string; color?: string; action: string }>;
  }, [buttons]);

  const handleClick = useCallback((action: string) => {
    if (disabled || loading) return;
    setClickedAction(action);
    onButtonClick();
  }, [disabled, loading, setClickedAction, onButtonClick]);

  const justifyMap: Record<string, string> = {
    left: 'flex-start', center: 'center', right: 'flex-end', stretch: 'stretch',
  };

  return (
    <div style={{
      width: '100%', height: '100%', display: 'flex', alignItems: 'center',
      justifyContent: justifyMap[alignment] || 'flex-end', gap: '8px',
      padding: '0 8px', boxSizing: 'border-box',
      fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
    }}>
      <style>{`@keyframes spin { to { transform: rotate(360deg); } }`}</style>
      {safeButtons.map((btn, i) => (
        <button
          key={i}
          onClick={() => handleClick(btn.action)}
          disabled={disabled || loading}
          style={{
            padding: '6px 14px', fontSize: '13px', fontWeight: 500,
            borderRadius: '6px', border: 'none', cursor: disabled ? 'default' : 'pointer',
            backgroundColor: btn.color || '#3D7FFF', color: '#fff',
            opacity: disabled ? 0.5 : 1, flex: alignment === 'stretch' ? 1 : undefined,
            display: 'flex', alignItems: 'center', justifyContent: 'center', gap: '6px',
          }}
        >
          {loading && (
            <div style={{
              width: 12, height: 12, border: '2px solid rgba(255,255,255,0.3)',
              borderTopColor: '#fff', borderRadius: '50%',
              animation: 'spin 0.6s linear infinite',
            }} />
          )}
          {btn.label}
        </button>
      ))}
    </div>
  );
};
```

### Usage in Retool:
- **Event handler on `buttonClick`:** Run script:
  ```js
  if (ActionButtonGroup1.clickedAction === "approve") {
    approveQuery.trigger();
  } else if (ActionButtonGroup1.clickedAction === "reject") {
    rejectQuery.trigger();
  }
  ```

---

## Template: Minimal Starter

Use this as a quick starting point for any new component:

```tsx
import React, { FC, useMemo, useCallback } from 'react';
import { Retool } from '@tryretool/custom-component-support';

export const MyComponent: FC = () => {
  Retool.useComponentSettings({ defaultHeight: 30, defaultWidth: 6 });

  // ── Inspector Properties ──
  const [data] = Retool.useStateArray({
    name: "data", initialValue: [], label: "Data",
    description: "Bind to {{ yourQuery.data }}",
  });
  const [loading] = Retool.useStateBoolean({
    name: "loading", initialValue: false, label: "Loading", inspector: "text",
    description: "Bind to {{ yourQuery.isFetching }}",
  });
  const [disabled] = Retool.useStateBoolean({
    name: "disabled", initialValue: false, label: "Disabled", inspector: "text",
  });

  // ── Output ──
  const [selectedItem, setSelectedItem] = Retool.useStateObject({
    name: "selectedItem", initialValue: {}, inspector: "hidden",
  });

  // ── Events ──
  const onClick = Retool.useEventCallback({ name: "click" });

  // ── Logic ──
  const safeData = useMemo(() => Array.isArray(data) ? data : [], [data]);

  const handleClick = useCallback((item: any) => {
    if (disabled) return;
    setSelectedItem(item);
    onClick();
  }, [disabled, setSelectedItem, onClick]);

  // ── Render ──
  if (loading) {
    return (
      <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'center', height: '100%' }}>
        <span>Loading...</span>
      </div>
    );
  }

  if (safeData.length === 0) {
    return (
      <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'center', height: '100%', color: '#9CA3AF' }}>
        No data
      </div>
    );
  }

  return (
    <div style={{
      width: '100%', height: '100%', overflow: 'auto',
      fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
    }}>
      {safeData.map((item, i) => (
        <div key={i} onClick={() => handleClick(item)} style={{ padding: '8px 12px', cursor: 'pointer' }}>
          {JSON.stringify(item)}
        </div>
      ))}
    </div>
  );
};
```

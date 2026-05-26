# Repeater Pagination — Approach Comparison

**Created:** 2026-05-26
**Purpose:** Document both old and new pagination approaches so we can switch back if needed.

---

## Approach A — Original (Toggle + Items Per Load + Infinite Scroll controls)

**Status:** Replaced in prototype on 2026-05-26

### Settings Panel (Repeater Inspector)

The Pagination section in the repeater inspector contained:

1. **Infinite Scroll toggle** — ON/OFF switch that explicitly controls whether the repeater uses infinite scroll or loads all items at once.
2. **Items per load** (read-only) — Displayed the current page size value from the context config. Shown when pagination is ON.
3. **"Edit in [Context name] configuration →"** link — Navigated to the context config panel to change the page size.
4. **Off-state note** — When pagination is OFF, a note explained: "Without pagination, all items load at once. To control how many items are shown, set the item limit in the context configuration →."

### Settings Panel (Context Config — `renderItemsParams`)

The Items parameters section contained:

1. **Sort** — dropdown with sort field options
2. **Filter** — add/remove filter rules
3. **Page size** — editable number input (1–100)
4. **Pagination** — segmented control: "Infinite scroll" | "Load more"
5. **Show limit** — toggle with slider (1–100 for scroll, 1–1000 for button)

### Behavior

- Pagination mode was explicitly toggled by the builder via a toggle (inspector) or segmented control (context config).
- `mode` values: `'scroll'` (infinite scroll), `'button'` (load more), `'none'` (no pagination).
- Items per load was configured in context config and shown read-only in the inspector.
- The Infinite Scroll toggle and the Items per load row were always visible when Items was bound (in the inspector pagination section).

### State shape

```js
S.repPagination[key] = {
  mode: 'scroll',        // 'scroll' | 'button' | 'none'
  showLimit: false,
  limit: 36,
  itemsPerLoad: 12,
  infoBannerDismissed: false,
};
```

### Product spec reference

Full spec: `artifacts/product/product-spec-repeater-pagination.md`

Key FRs:
- FR-001: Toggle pagination mode (ON/OFF) — explicit Infinite Scroll toggle
- FR-002: Show items per load as read-only in inspector
- FR-003: Off-state note when pagination is OFF
- FR-004: Configure items per load / item limit in context config (slider 1–100)
- FR-005: Performance warning for high item counts (>50)
- FR-006: Hide pagination section when Items is unbound
- FR-007: Sort and filter navigation hint

---

## Approach B — New (Load More function picker, no explicit pagination controls)

**Status:** Active in prototype as of 2026-05-26

### Settings Panel (Repeater Inspector)

The Pagination section is simplified:

1. **ArrayItems controller** — the existing Items binding (unchanged).
2. **Load More function picker** — a new function binding row that lets the builder pick a "Load More" function from the context.
3. No "Items per load" row.
4. No "Infinite Scroll" toggle.

### Behavior

- **No explicit "Items per Load" or "Infinite Scroll" controls** in the Settings Panel.
- These settings are **derived from the context** — the panel only displays the ArrayItems controller and the Load More function picker.
- The **presence of a bound "Load More" function** determines whether the repeater supports infinite scroll, replacing the former Infinite Scroll toggle.
  - Load More function bound → repeater has infinite scroll / progressive loading.
  - Load More function not bound → repeater loads all items at once (no pagination).
- Page size / items per load is implicit in the loading function's implementation, not configured via a UI control.

### Key differences from Approach A

| Aspect | Approach A (Old) | Approach B (New) |
|--------|-----------------|-----------------|
| Infinite scroll control | Explicit toggle in inspector | Implicit — determined by presence of Load More function |
| Items per load | Read-only value in inspector, editable in context config | Not shown — derived from context/function |
| Load More function | Not exposed in settings panel | Function picker in settings panel |
| Pagination segmented control | "Infinite scroll" / "Load more" in context config | Removed |
| Builder mental model | "I toggle pagination on/off and configure batch size" | "I connect a loading function — if it exists, items load progressively" |

---

## How to revert to Approach A

1. Restore the Infinite Scroll toggle in both `renderSettingsRepeaterOnRepeater()` and `renderSettings()` pagination sections.
2. Restore the "Items per load" read-only row and "Edit in [Context name] configuration →" link.
3. Restore the off-state note for pagination OFF.
4. In `renderItemsParams()`, restore the "Pagination" segmented control and "Page size" input.
5. Remove the "Load More" function picker row from the inspector.
6. The git history has the full old implementation — check commits before 2026-05-26 on this branch.

# Product Spec — Repeater Inspector: Pagination (Phase 1)

**Version:** 1.0  
**Date:** 2026-05-03  
**Scope:** Pagination section behavior in the Repeater inspector and linked context configuration panel — all use cases

---

## Table of Contents

1. [Overview](#overview)
2. [Terminology](#terminology)
3. [Intent Hierarchy](#intent-hierarchy)
4. [Requirements Summary](#requirements-summary)
5. [Functional Requirements](#functional-requirements)
   - [I1 — Control how items load for site visitors](#i1--control-how-items-load-for-site-visitors)
     - [FR-001 Toggle pagination mode (ON / OFF)](#fr-001-toggle-pagination-mode-on--off)
     - [FR-006 Hide pagination section when Items is unbound](#fr-006-hide-pagination-section-when-items-is-unbound)
   - [I2 — Configure the number of items per batch or total](#i2--configure-the-number-of-items-per-batch-or-total)
     - [FR-002 Show items per load as read-only in inspector](#fr-002-show-items-per-load-as-read-only-in-inspector-pagination-on)
     - [FR-003 Inform builder about item limit path when pagination is OFF](#fr-003-inform-builder-about-item-limit-path-when-pagination-is-off)
     - [FR-004 Configure items per load / item limit in context config](#fr-004-configure-items-per-load--item-limit-in-context-config)
     - [FR-007 Sort and filter navigation hint in Properties](#fr-007-sort-and-filter-navigation-hint-in-properties)
   - [I3 — Understand performance impact](#i3--understand-performance-impact)
     - [FR-005 Performance warning for high item counts](#fr-005-performance-warning-for-high-item-counts)
6. [Non-Functional Requirements](#non-functional-requirements)
   - [Intent-Tied NFRs](#intent-tied-nfrs)
   - [Cross-Cutting NFRs](#cross-cutting-nfrs)

---

## Overview

### Problem to Solve

Builders connecting a Repeater to data need a clear, guided way to control how many items load — whether progressively via infinite scroll or all at once — without being overwhelmed by configuration they don't own. The items per load and item limit settings live on the context, not the repeater itself, and the inspector must surface this boundary clearly so builders always know where to act.

### Inputs Used

- Prototype: `pages/context-binding-prototype/binding-platform-demo.html` (primary)
- Blank Repeater PRD: `studio2-cms-prototype/blank-repeater-prd.md` (supporting context)

---

## Terminology

| Term | Definition |
|------|-----------|
| **Repeater** | Editor element that repeats a layout for each item in a bound data array |
| **Context** | A data source instance connected to a Repeater; exposes fields for binding |
| **Context config panel** | The panel where context-level settings (page size, sort, filter) are configured |
| **Items** | The Repeater's data binding property — binds to an array field from a context |
| **Pagination** | The feature that controls whether items load progressively (infinite scroll) or all at once |
| **Infinite scroll** | Pagination mode where new items load automatically as the visitor scrolls |
| **Items per load** | Number of items fetched per scroll batch (label when pagination is ON) |
| **Item limit** | Maximum number of items shown at once (label when pagination is OFF) |
| **Page size** | Underlying data parameter name for both items per load and item limit (stored on the context) |
| **Builder** | Wix Studio user configuring the Repeater (site creator or web professional) |
| **Visitor** | End user of the published site |

---

## Intent Hierarchy

| ID | Main Intent | Sub-Intents | Current feeling | Expected feeling |
|----|-------------|-------------|----------------|-----------------|
| **I1** | Control how items load for site visitors | I1a — Enable infinite scroll · I1b — Disable pagination | Uncertain — not sure whether the repeater is paginating or loading everything | In control — immediately clear what mode is active |
| **I2** | Configure the number of items per batch or total | I2a — Set items per load (pagination ON) · I2b — Set item limit (pagination OFF) | Confused — unsure where to change this value | Guided — knows the value lives in context config and gets there in one click |
| **I3** | Understand performance impact of their settings | I3a — Get warned when configured value risks slow load times | Unaware — would unknowingly publish something slow | Informed — warned before it becomes a problem |

---

## Requirements Summary

| FR ID | Title | Use Cases | Linked Intent | Phase | Priority |
|-------|-------|-----------|--------------|-------|----------|
| FR-001 | Toggle pagination mode (ON / OFF) | UC-001a, UC-001b, UC-001c | I1, I1a, I1b | 1 | Blocker |
| FR-002 | Show items per load as read-only in inspector | UC-002a, UC-002b | I2, I2a | 1 | Blocker |
| FR-003 | Inform builder about item limit path when pagination is OFF | UC-003a, UC-003b | I1b, I2b | 1 | Blocker |
| FR-004 | Configure items per load / item limit in context config | UC-004a, UC-004b, UC-004c | I2a, I2b | 1 | Blocker |
| FR-005 | Performance warning for high item counts | UC-005a, UC-005b | I3, I3a | 1 | Must have |
| FR-006 | Hide pagination section when Items is unbound | UC-006a, UC-006b | I1 | 1 | Blocker |
| FR-007 | Sort and filter navigation hint in Properties | UC-007a, UC-007b, UC-007c, UC-007d | I2 | 1 | Must have |

---

## Functional Requirements

---

### I1 — Control how items load for site visitors

---

#### FR-001 Toggle pagination mode (ON / OFF)

**Description:** The Repeater inspector exposes a toggle that allows the builder to switch between pagination ON (infinite scroll) and pagination OFF (load all items at once). The toggle is only visible when the Repeater's Items property is bound to a context.

**Scope:** Phase 1  
**Priority:** Blocker

---

##### UC-001a — Enable infinite scroll

**Use Case Goal:** Builder turns on progressive item loading so visitors load items in batches as they scroll.

**Use Case Description:** A builder with a connected Repeater wants visitors to see items load progressively rather than waiting for the entire dataset to load at once. They activate the infinite scroll toggle in the Pagination section.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with Items bound and pagination currently OFF, then opens the Pagination section in the inspector.

**Happy Flow:**
1. Builder selects a Repeater with Items bound; pagination is currently OFF.
2. Inspector renders the Pagination section with the toggle in the OFF state.
3. Builder activates the toggle.
4. Toggle moves to the ON (infinite scroll) state.
5. Inspector replaces the off-state note with the "Items per load" read-only row, showing the current value from the context config (default: 25).
6. "Edit in [Context name] configuration →" link appears below the value.
7. Sort/filter navigation hint appears in the Properties section below the Items binding chip.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Concurrent / State Changes | Important | Multiple Repeaters share the same context | Activating pagination on one Repeater does not change the page size on the shared context; only the mode of the selected Repeater changes |
| Incomplete / Partial States | Important | Context is bound but context config panel is unavailable or errored | Toggle still renders and operates; the "Edit in [Context name] configuration →" link is present but the target panel may show an empty or error state |
| Boundary Conditions | Critical | Context config has no page size value set | System uses the default value of 25 and displays it in the read-only row |

---

##### UC-001b — Disable pagination

**Use Case Goal:** Builder switches the Repeater to load all items at once, without scroll-triggered loading.

**Use Case Description:** A builder wants the Repeater to render a complete, static list — for example, a curated set of featured products where all items should be visible immediately. They turn off infinite scroll.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with Items bound and pagination currently ON, then opens the Pagination section.

**Happy Flow:**
1. Builder selects a Repeater with Items bound; pagination is currently ON.
2. Inspector shows toggle in ON state, "Items per load" read-only value, and "Edit in [Context name] configuration →" link.
3. Builder deactivates the toggle.
4. Toggle moves to the OFF state.
5. "Items per load" row and edit link are replaced by the off-state note.
6. Note reads: "Without pagination, all items load at once. To control how many items are shown, set the item limit in the [context configuration →]."
7. "context configuration →" is a clickable link that opens the context config panel.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Boundary Conditions | Critical | Builder toggles OFF then immediately back ON | State toggles cleanly; no stale note remains; read-only value restores correctly |
| Error States | Important | Context is removed while pagination is ON | Pagination section is hidden (FR-006 handles unbound state); no dangling toggle |

---

##### UC-001c — Default pagination state when Items first binds

**Use Case Goal:** System sets a performant default so the Repeater works well out of the box without manual configuration.

**Use Case Description:** When a builder binds Items for the first time, the system initialises pagination as ON (infinite scroll) with a default page size of 25, balancing performance and content visibility.

**Actor:** System  
**Trigger:** Items binding is created on a Repeater for the first time.

**Happy Flow:**
1. Builder binds Items on an unbound Repeater.
2. System initialises pagination state as ON (infinite scroll) with page size = 25.
3. Inspector opens on the Repeater with the Pagination section visible, toggle in ON state, and items per load showing 25.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Boundary Conditions | Critical | Items is re-bound to a different context after being disconnected | Previously set pagination state and page size are preserved; they are not reset to defaults |
| Incomplete / Partial States | Important | Binding is created but no context config values exist yet | System falls back to default (25) for the displayed read-only value |

---

##### Codebase Implications — FR-001

- **Pagination state storage:** Per-repeater pagination mode is stored in `S.repPagination[key]`, keyed by repeater ID. `getRepPag(repId)` initialises the state with `{ mode: 'scroll', itemsPerLoad: 12, showLimit: false, limit: 36 }` if not already present. Default `itemsPerLoad` and `limit` values should be aligned to 25.
- **Mode values:** `mode: 'scroll'` = pagination ON (infinite scroll); `mode: 'none'` = pagination OFF. `setRepPagMode(repId, mode)` handles the toggle.
- **Inspector render paths:** Two functions render the Pagination section: `renderSettingsRepeaterOnRepeater()` (nested repeaters) and `renderSettings()` (standard repeaters). Both must be kept in sync for any change to the toggle behavior.
- **Context dependency:** The read-only items per load value reads `S.cfgValues?.[ctxId]?.pageSize`. Context ID is resolved from `S.bindings[repId + '|Items'].ctxId`.
- **Permission / PII:** Pagination mode is a builder-level setting; no PII implications.

---

#### FR-006 Hide pagination section when Items is unbound

**Description:** The Pagination section in the Repeater inspector is conditionally rendered. It is only shown when the Repeater has an active Items binding. When Items is unbound, the entire section is absent.

**Scope:** Phase 1  
**Priority:** Blocker

---

##### UC-006a — Pagination section absent for unbound repeater

**Use Case Goal:** Builder does not encounter pagination controls that have no effect, keeping the inspector focused.

**Use Case Description:** A builder selects a blank or disconnected Repeater. Since there is no data source, the Pagination section serves no purpose and must not appear.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with no Items binding.

**Happy Flow:**
1. Builder selects a Repeater with no Items binding.
2. Inspector shows the Properties section with "Connect to an array field →" prompt.
3. Pagination section is not rendered anywhere in the inspector.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Empty States | Critical | Repeater has a context attached but Items is not yet bound | Pagination section remains hidden — a context attachment alone does not reveal it |

---

##### UC-006b — Pagination section disappears when Items is disconnected

**Use Case Goal:** Pagination controls are immediately hidden when the builder removes the Items binding, preventing controls from appearing in an invalid state.

**Use Case Description:** A builder disconnects Items from a previously bound Repeater. The Pagination section should disappear immediately.

**Actor:** Builder  
**Trigger:** Builder disconnects the Items binding.

**Happy Flow:**
1. Builder selects a bound Repeater; Pagination section is visible.
2. Builder disconnects the Items binding.
3. Inspector updates; Pagination section is no longer rendered.
4. Pagination state for the repeater is preserved internally (not reset), so re-binding Items restores the previous mode.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Concurrent / State Changes | Important | Items is disconnected while context config panel is open | Context config panel closes or becomes unreachable; no orphaned controls remain visible |

---

##### Codebase Implications — FR-006

- **Render condition:** The Pagination section is gated on `S.bindings[repId + '|Items']` being truthy. Both `renderSettingsRepeaterOnRepeater()` and `renderSettings()` must apply this gate.
- **State preservation:** Pagination state in `S.repPagination` is not cleared on disconnect — it is preserved so that re-binding Items restores the previous configuration.

---

### I2 — Configure the number of items per batch or total

---

#### FR-002 Show items per load as read-only in inspector (pagination ON)

**Description:** When pagination is ON, the Repeater inspector displays the current "Items per load" value as a read-only field. The value is sourced from the context config (`pageSize`). An "Edit in [Context name] configuration →" link provides direct navigation to the context config panel for editing. No editable control is present in the inspector itself.

**Scope:** Phase 1  
**Priority:** Blocker

---

##### UC-002a — View current items per load value

**Use Case Goal:** Builder can see the current items per load setting at a glance without opening the context config panel.

**Use Case Description:** A builder reviews their Repeater configuration and wants a quick reference to the current items per load value before deciding whether to adjust it.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with Items bound and pagination ON.

**Happy Flow:**
1. Builder selects Repeater with Items bound and pagination ON.
2. Inspector renders the Pagination section.
3. "Items per load" row displays the current value from the context config (default: 25 if not explicitly set).
4. Value is presented as read-only — no editable input or slider is present in the inspector.
5. Tooltip on the "?" icon reads: "How many items load at a time as visitors scroll. Use a lower number for faster loading, or a higher number to load more items at once."
6. "Edit in [Context name] configuration →" link appears below the value.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Error States | Critical | Context config has no page size value set | Inspector displays the default value (25) |
| Concurrent / State Changes | Important | Page size is changed from the context config panel while inspector is visible | Inspector read-only value updates in real time to reflect the new value without a manual refresh |
| Boundary Conditions | Important | Value is at minimum (1) or maximum (100) | Inspector renders the value correctly without truncation or overflow |

---

##### UC-002b — Navigate to context config to edit

**Use Case Goal:** Builder reaches the context config panel to change the items per load value without navigating away from the inspector flow.

**Use Case Description:** A builder sees the read-only value and wants to adjust it. They click the direct link to open the context config panel.

**Actor:** Builder  
**Trigger:** Builder clicks "Edit in [Context name] configuration →" in the Pagination section.

**Happy Flow:**
1. Builder clicks the "Edit in [Context name] configuration →" link.
2. System opens the context configuration panel for the bound context.
3. Context config panel shows the "Items per load" control with slider and number input.
4. Current value matches what was shown in the inspector.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Error States | Important | Context has been removed after the binding was set | Link is present but context config panel renders an error or empty state; system does not crash |

---

##### Codebase Implications — FR-002

- **Value source:** Read-only value reads `S.cfgValues?.[ctxId]?.pageSize || '25'`. Falls back to `'25'` when not set.
- **Context name resolution:** Link label uses `resolveCtxName(ctx.id)` to display the human-readable context name dynamically.
- **Navigation target:** Clicking the link sets `S.ctxSettingsView = ctxId` and triggers `renderBody()` to open the context config panel.
- **Real-time sync:** Because both the inspector and context config panel read from the same `S.cfgValues` object, any change in context config is immediately reflected in the inspector on next render.

---

#### FR-003 Inform builder about item limit path when pagination is OFF

**Description:** When pagination is OFF, the inspector replaces the items per load row with a contextual note. The note explains that all items load at once and provides a direct link to the context config panel where the item limit can be controlled.

**Scope:** Phase 1  
**Priority:** Blocker

---

##### UC-003a — View contextual note about all-items-load behavior

**Use Case Goal:** Builder understands the consequence of disabling pagination before publishing, without having to discover it through testing.

**Use Case Description:** A builder has disabled pagination. The inspector surfaces a clear explanation of what this means for site visitors so the builder can make an informed decision.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with Items bound and pagination OFF.

**Happy Flow:**
1. Builder selects Repeater with Items bound and pagination OFF.
2. Inspector shows toggle in OFF state.
3. Below the toggle, a contextual note reads: "Without pagination, all items load at once. To control how many items are shown, set the item limit in the [context configuration →]."
4. "context configuration →" is rendered as a clickable link.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Loading States | Nice-to-have | Context has many records and the builder is warned about potential slow loads | The off-state note alone is sufficient context; the performance warning at >50 (FR-005) is shown in context config, not here |

---

##### UC-003b — Navigate to context config from the off-state note

**Use Case Goal:** Builder reaches context config directly from the pagination-off note to set an appropriate item limit.

**Use Case Description:** A builder reads the note and decides they want to cap the number of items shown. They click the link to go straight to context config.

**Actor:** Builder  
**Trigger:** Builder clicks "context configuration →" in the off-state note.

**Happy Flow:**
1. Builder clicks "context configuration →" in the note.
2. System opens the context configuration panel for the bound context.
3. Panel shows "Item limit" label (not "Items per load") since pagination is OFF.
4. Slider and number input show the current item limit value (default: 25).

---

##### Codebase Implications — FR-003

- **Conditional render:** The off-state note is shown when `p.mode !== 'scroll'` (i.e., pagination is OFF). The ON-state read-only row and the OFF-state note are mutually exclusive branches in both inspector render functions.
- **Navigation:** Same navigation mechanism as FR-002 — sets `S.ctxSettingsView = ctxId` on click.

---

#### FR-004 Configure items per load / item limit in context config

**Description:** The context configuration panel exposes a slider (range 1–100) and a synced number input for setting the page size value. The label and tooltip adapt based on the Repeater's current pagination state: "Items per load" when pagination is ON, "Item limit" when pagination is OFF. The default value is 25.

**Scope:** Phase 1  
**Priority:** Blocker

---

##### UC-004a — Set items per load when pagination is ON

**Use Case Goal:** Builder tunes the fetch batch size to balance load speed and content density for visitors.

**Use Case Description:** A builder with infinite scroll enabled wants to control how many items load per scroll batch. They open the context config panel and adjust the value.

**Actor:** Builder  
**Trigger:** Builder opens the context config panel while the bound Repeater's pagination is ON.

**Happy Flow:**
1. Builder opens the context config panel (via "Edit in [Context name] configuration →" or directly).
2. Panel displays "Items per load" label with tooltip: "How many items load at a time as visitors scroll. Use a lower number for faster loading, or a higher number to load more items at once."
3. Slider (1–100) and number input are visible and synced to the current value (default: 25).
4. Builder adjusts the slider or enters a value in the number input.
5. Both controls update to reflect the new value in real time.
6. The read-only "Items per load" value in the inspector updates to match.
7. If the new value exceeds 50, the performance warning appears inline (FR-005).

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Validation / Input Constraints | Critical | Builder enters a non-numeric value or leaves the field empty | System clamps to last valid value or reverts to default (25) |
| Boundary Conditions | Critical | Builder sets value to 1 (minimum) | Slider and input accept the value; no warning shown |
| Boundary Conditions | Critical | Builder sets value to 100 (maximum) | Slider and input accept the value; performance warning is shown (threshold > 50) |
| Boundary Conditions | Critical | Builder sets value to exactly 50 | No performance warning shown |
| Boundary Conditions | Critical | Builder sets value to 51 | Performance warning appears |
| Concurrent / State Changes | Important | Multiple Repeaters use the same context | Page size change in context config applies to all Repeaters consuming that context; this is expected behavior |

---

##### UC-004b — Set item limit when pagination is OFF

**Use Case Goal:** Builder caps the total number of items the Repeater renders, so not all collection items load at once.

**Use Case Description:** A builder has disabled pagination and wants to limit the repeater to, for example, the 6 most recent blog posts rather than the entire collection.

**Actor:** Builder  
**Trigger:** Builder opens the context config panel while the bound Repeater's pagination is OFF.

**Happy Flow:**
1. Builder opens the context config panel.
2. Panel displays "Item limit" label with tooltip: "Maximum number of items the repeater displays. Since pagination is off, all items load at once — keep this number low to avoid slow load times."
3. Slider (1–100) and number input show the current value (default: 25).
4. Builder adjusts the value.
5. Both controls update in real time.
6. If the new value exceeds 50, the performance warning appears inline (FR-005).

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Validation / Input Constraints | Critical | Builder enters 0 or a negative number | System clamps to 1 (minimum valid value) |
| Validation / Input Constraints | Critical | Builder enters a value above 100 | System clamps to 100 (maximum) |
| Boundary Conditions | Important | Collection has fewer items than the item limit | Repeater renders only the available items; the limit acts as a ceiling, not a floor |

---

##### UC-004c — Label and tooltip adapt when pagination state changes

**Use Case Goal:** Builder always sees a label and tooltip that accurately reflect the current semantic meaning of the setting.

**Use Case Description:** The same underlying page size value serves two different purposes depending on pagination state. The context config panel must always display the correct label and tooltip so the builder is not confused about what they are configuring.

**Actor:** System  
**Trigger:** Repeater's pagination state changes (toggle ON → OFF or OFF → ON), and the context config panel is next opened or re-rendered.

**Happy Flow:**
1. Builder changes the pagination toggle state on the Repeater inspector.
2. Context config panel is opened (or was already open and re-renders).
3. Label updates: "Items per load" ↔ "Item limit" based on current pagination mode.
4. Tooltip updates to match the new label's meaning.
5. Slider and number input value remain unchanged.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Concurrent / State Changes | Nice-to-have | Pagination state toggled while context config panel is open mid-edit | On next render, label and tooltip reflect the new state; in-progress edits to the value are preserved |

---

##### Codebase Implications — FR-004

- **Label resolution:** `_psLabel = _pagOff ? 'Item limit' : 'Items per load'`. `_pagOff` is derived from `getRepPag(S.sel)?.mode !== 'scroll'`.
- **Tooltip resolution:** `_psTip` switches between the two tooltip strings based on `_pagOff`.
- **Value storage:** Page size is stored at `S.cfgValues[ctxId].pageSize`. Both the slider `oninput` and number input `onchange` handlers write to this path and call `renderBody()`.
- **Two render paths:** Context config is rendered in both a flat mode and an accordion/multi-level mode inside `renderCtxSettings(ctxId)`. Both paths use the same `pageSizeRow` variable and must reflect the same label/tooltip logic.
- **Default:** Both render paths fall back to `'25'` if `pageSize` is not set.
- **Slider/input sync:** The slider and number input share the same `_psVal` / `_psVal2` variable. Changing one re-renders the component, updating the other.

---

#### FR-007 Sort and filter navigation hint in Properties

**Description:** When the Repeater's Items property is bound, the Properties section below the Items binding chip displays sort and filter status in one of two states:

- **No rules configured:** A discovery hint reads "Want to sort/filter items? Go to [Context name] configuration →".
- **Rules configured:** Active sort and filter rule counts are shown as inline summary chips (e.g., "↓ 2 sort rules", "⚑ 2 filters"), followed by an "Edit in [Context name] configuration →" link.

Both states use the context's display name in the link and navigate to the context config panel on click.

**Scope:** Phase 1  
**Priority:** Must have

---

##### UC-007a — View sort/filter discovery hint (no rules configured)

**Use Case Goal:** Builder discovers that sort and filter configuration lives in context config without having to search for it.

**Use Case Description:** A builder sees their Items binding chip in the Properties section with no sort or filter rules set on the context. A hint below the chip indicates where to go.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with Items bound and no sort or filter rules configured on the context.

**Happy Flow:**
1. Builder selects a Repeater with Items bound.
2. Properties section shows the Items binding chip.
3. Below the chip, the hint reads: "Want to sort/filter items? Go to [Context name] [configuration →]."
4. "[configuration →]" is rendered as a clickable link.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Empty States | Critical | Items is not bound | Hint is not shown; Properties shows the unbound state prompt instead |
| Boundary Conditions | Important | Context name is very long | Context name truncates or wraps gracefully; the link remains clickable |

---

##### UC-007b — Navigate to context config via discovery hint

**Use Case Goal:** Builder reaches the context config panel to configure sort or filter in one click from the Properties section.

**Actor:** Builder  
**Trigger:** Builder clicks the "[configuration →]" link in the discovery hint.

**Happy Flow:**
1. Builder clicks "[configuration →]" link.
2. System opens the context configuration panel for the bound context.
3. Panel shows sort and filter controls alongside the items per load / item limit control.

---

##### UC-007c — View active sort and filter summary chips

**Use Case Goal:** Builder sees at a glance that sort and filter rules are already configured on the context, and how many are active.

**Use Case Description:** A builder selects a Repeater whose bound context already has sort or filter rules set. Instead of the discovery hint, the Properties section shows compact summary chips that convey the count of active rules — giving the builder immediate confirmation that the context is configured without requiring them to open the config panel.

**Actor:** Builder  
**Trigger:** Builder selects a Repeater with Items bound and at least one sort rule or filter rule configured on the context.

**Happy Flow:**
1. Builder selects a Repeater with Items bound and active sort/filter rules on the context.
2. Properties section shows the Items binding chip.
3. Below the chip, one or both summary chips are shown:
   - Sort chip: "↓ [n] sort rules" (shown when ≥ 1 sort rule is configured)
   - Filter chip: "⚑ [n] filters" (shown when ≥ 1 filter is configured)
4. Below the chips, "Edit in [Context name] configuration →" link is shown.
5. Discovery hint ("Want to sort/filter items?") is not shown.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Boundary Conditions | Critical | Only sort rules are configured, no filters | Only the sort chip is shown; filter chip is absent |
| Boundary Conditions | Critical | Only filters are configured, no sort rules | Only the filter chip is shown; sort chip is absent |
| Boundary Conditions | Critical | Exactly 1 sort rule | Chip reads "↓ 1 sort rule" (singular) |
| Boundary Conditions | Critical | Exactly 1 filter | Chip reads "⚑ 1 filter" (singular) |
| Concurrent / State Changes | Important | All sort and filter rules are removed from the context while the inspector is open | Summary chips disappear; discovery hint reappears on next render |

---

##### UC-007d — Navigate to context config via active summary chips

**Use Case Goal:** Builder reaches the context config panel to review or edit existing sort and filter rules in one click.

**Actor:** Builder  
**Trigger:** Builder clicks "Edit in [Context name] configuration →" below the active summary chips.

**Happy Flow:**
1. Builder clicks "Edit in [Context name] configuration →".
2. System opens the context configuration panel for the bound context.
3. Panel shows the currently configured sort and filter rules, and the items per load / item limit control.

---

##### Codebase Implications — FR-007

- **State branching:** The Properties section has two mutually exclusive display states gated on whether the context has any configured sort or filter values. When `S.cfgValues?.[ctxId]?.sort` or any active filter is set, show summary chips + edit link. Otherwise show the discovery hint.
- **Chip content:** Sort chip label reads "↓ [n] sort rule(s)"; filter chip reads "⚑ [n] filter(s)". Singular/plural must be handled.
- **Render condition:** Both states render only when `b && ctx` are truthy (Items is bound and context resolved). Uses `resolveCtxName(ctx.id)` for all link labels.
- **Navigation:** `onclick="S.ctxSettingsView='${ctx.id}';renderBody()"` — same pattern as all other context config navigation links.
- **Both inspector paths:** Both display states must appear in `renderSettingsRepeaterOnRepeater()` and `renderSettings()`.

---

### I3 — Understand performance impact

---

#### FR-005 Performance warning for high item counts

**Description:** When the page size value in the context config panel exceeds 50, an inline amber warning is displayed below the slider and number input. The warning is advisory — it does not block saving or prevent the builder from publishing. It clears immediately when the value drops to 50 or below.

**Scope:** Phase 1  
**Priority:** Must have

---

##### UC-005a — Warning appears when value exceeds 50

**Use Case Goal:** Builder is notified of a potential performance risk before publishing, while remaining able to keep the high value if intentional.

**Use Case Description:** A builder adjusts the items per load or item limit value upward past 50. The system surfaces an inline warning so they are aware of the potential impact on site load speed, without preventing them from proceeding.

**Actor:** Builder  
**Trigger:** Builder sets the page size value to 51 or higher in the context config panel.

**Happy Flow:**
1. Builder sets value to 51 or above (via slider or number input).
2. Warning appears inline below the controls: "Loading may be slow due to the high number of items loading at once."
3. Warning is styled in amber (caution, not error).
4. Builder can acknowledge and keep the value, or lower it to clear the warning.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Boundary Conditions | Critical | Value is exactly 50 | No warning shown |
| Boundary Conditions | Critical | Value is exactly 51 | Warning appears |
| Boundary Conditions | Critical | Value is 100 (maximum) | Warning is shown; the warning does not impose an upper limit — 100 is still a valid value |
| Error States | Important | Builder closes context config with value > 50 and reopens it | Warning is shown immediately on open since the value still exceeds the threshold |

---

##### UC-005b — Warning clears when value is lowered

**Use Case Goal:** Builder receives immediate positive feedback when they resolve the performance concern.

**Use Case Description:** A builder sees the warning and decides to lower the value. The warning disappears as soon as the value drops to 50 or below.

**Actor:** Builder  
**Trigger:** Builder lowers the page size value to ≤ 50 while the warning is visible.

**Happy Flow:**
1. Warning is visible (current value > 50).
2. Builder adjusts slider or number input to 50 or below.
3. Warning disappears immediately.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|----------|----------|---------|--------------------------|
| Boundary Conditions | Critical | Builder drags slider from above 50 down through 50 | Warning disappears the moment value crosses to ≤ 50; no lag or residual warning visible |

---

##### Codebase Implications — FR-005

- **Threshold check:** Warning renders when `_psVal > 50`. This conditional is evaluated on every render of the `pageSizeRow` component.
- **Warning copy:** "Loading may be slow due to the high number of items loading at once."
- **Styling:** Amber / warning visual treatment — not red (error). Non-blocking; no save or publish gate.
- **Both context config render paths:** Warning must be present in both the flat-mode and accordion-mode rendering of `pageSizeRow` inside `renderCtxSettings(ctxId)`.

---

## Non-Functional Requirements

### Intent-Tied NFRs

| NFR ID | Category | Description | Threshold | Impl Status | Scope | Linked Intents |
|--------|----------|-------------|-----------|-------------|-------|----------------|
| NFR-I1-01 | Usability | Toggle state change is reflected in the inspector without perceptible delay | ≤ 200ms from interaction to visual update | [NEW] | Delta-only | I1 |
| NFR-I1-02 | Usability | Pagination section renders identically in both inspector render paths (standard repeater and repeater-on-repeater) | Zero visual or functional divergence between the two paths | [NEW] | Delta-only | I1 |
| NFR-I2-01 | Usability | Read-only items per load value in the inspector reflects the current context config value in real time | No stale value displayed after a context config change; sync on next render cycle | [NEW] | Delta-only | I2 |
| NFR-I2-02 | Usability | Slider and number input remain in sync — changing one updates the other without additional user action | Single user action (drag or type) updates both controls | [NEW] | Delta-only | I2 |
| NFR-I2-03 | Localization | Context name variable in "Edit in [Context name] configuration →" and sort/filter hint resolves correctly for all context types and name lengths | Dynamic string renders correctly for all supported context name lengths; no truncation of the interactive link | [NEW] | Delta-only | I2 |
| NFR-I3-01 | Error Messages | Performance warning is visually distinct from errors — amber styling, non-blocking copy | Warning uses amber (not red); copy does not imply failure; no save or publish gate | [NEW] | Delta-only | I3 |
| NFR-I3-02 | Usability | Warning appears and disappears in response to threshold crossing with no noticeable delay | ≤ 200ms from value change to warning appearing or clearing | [NEW] | Delta-only | I3 |

---

### Cross-Cutting NFRs

| NFR ID | Category | Description | Threshold | Impl Status | Scope | Linked Intents |
|--------|----------|-------------|-----------|-------------|-------|----------------|
| NFR-X-01 | Accessibility | Toggle, slider, and number input are keyboard operable and meet WCAG 2.1 AA requirements | Full keyboard operability; meets WCAG 2.1 AA contrast and interaction standards | [NEW] | Delta-only | I1, I2, I3 |
| NFR-X-02 | Localization | All labels, tooltips, and warning messages are localizable strings; no hard-coded display text | Zero hard-coded UI text; all strings externalized for translation | [NEW] | Delta-only | I1, I2, I3 |
| NFR-X-03 | Reliability | Pagination mode and page size value persisted per repeater — inspector reload shows correct state without reset | Zero regression on state after inspector close/reopen; state persists across re-renders of `renderBody()` | [NEW] | Delta-only | I1, I2 |
| NFR-X-04 | Supportability | Pagination state changes (toggle ON/OFF, page size value set) are observable in editor telemetry | Toggle events and page size change events emitted to telemetry on user interaction; no silent failures | [NEW] | Delta-only | I1, I2, I3 |

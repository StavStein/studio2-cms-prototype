# Product Spec: Repeater Event Handlers — Studio 2 Editor

**Version:** 1.0
**Date:** 2026-05-10
**Status:** Draft — pending platform team alignment on FR-002 and FR-005
**Consumers:** Engineering, UX, QA, Platform/Velo team

---

## Table of Contents

1. [Overview](#overview)
   - [Problem to Solve](#problem-to-solve)
   - [Target Audience](#target-audience)
   - [Intent Hierarchy](#intent-hierarchy)
2. [Requirements Summary](#requirements-summary)
3. [Functional Requirements](#functional-requirements)
   - [Intent 1: Wire Repeater lifecycle events from the Studio 2 Inspector](#intent-1)
     - [FR-001 \[MODIFY\]: Surface onItemReady and onItemRemoved in the Inspector](#fr-001)
     - [FR-002 \[NEW\]: Add onItemUpdated as a Repeater lifecycle event property](#fr-002)
   - [Intent 2: Wire item-level events with automatic per-item scoping](#intent-2)
     - [FR-003 \[NEW\]: Expose click and keyboard events on Repeater item elements](#fr-003)
     - [FR-004 \[NEW\]: Expose hover and mouse events on Repeater item elements](#fr-004)
   - [Intent 3: Hover and mouse events work reliably on dataset-connected Repeaters](#intent-3)
     - [FR-005 \[NEW\]: Ensure hover and mouse event delivery on dataset-connected Repeaters](#fr-005)
4. [Non-Functional Requirements](#non-functional-requirements)
   - [Intent-Tied NFRs](#intent-tied-nfrs)
   - [Cross-Cutting NFRs](#cross-cutting-nfrs)
5. [Open Dependencies](#open-dependencies)

---

## Overview

### Problem to Solve

The Repeater component in Wix Studio has an incomplete and partially broken event model: lifecycle events exist in the Velo API but are invisible in the Inspector, item-level event scoping silently fires across all items instead of the one interacted with, and hover events fail silently on dataset-connected Repeaters — the most common production configuration. Studio 2 must define a coherent, complete event contract for the Repeater component — exposing lifecycle events and item events directly from the Inspector via a function binding model — so developers and partners can build predictable, interactive Repeater experiences without writing manual boilerplate or workarounds.

### Target Audience

**Developers and Partners** (agencies and freelancers) building sites and apps with Studio 2 who need to add interactive or data-reactive behavior to Repeaters.

### Intent Hierarchy

| Intent | Description | Current feelings | Expected feelings | Priority |
|--------|-------------|-----------------|-------------------|----------|
| **Intent 1** | Wire Repeater lifecycle events from the Studio 2 Inspector | Frustrated, cut off — lifecycle event capability exists but is invisible from the Inspector | Empowered, capable — "I can see and control what this Repeater does, right where I'm designing it" | 1 |
| **Intent 2** | Wire item-level events with automatic per-item scoping | Confused, error-prone — events fire across all items; scoping workarounds are non-obvious and brittle | Confident, correct by default — correct scoping is the only available option from the panel | 1 |
| **Intent 3** | Hover and mouse events work reliably on dataset-connected Repeaters | Stuck, no signal — `onMouseIn`/`onMouseOut` silently stop working when a Repeater is dataset-connected | Predictable, reliable — hover/mouse events behave identically regardless of how the Repeater is populated | 2 |

---

## Requirements Summary

| FR ID | Title | Phase | Priority | Delta | Use Cases |
|-------|-------|-------|----------|-------|-----------|
| FR-001 | Surface onItemReady and onItemRemoved as event handler properties in the Inspector | 1 | Critical | `[MODIFY]` | UC-1.1: Attach handler when items load · UC-1.2: Attach handler when items are removed |
| FR-002 | Add onItemUpdated as a Repeater lifecycle event property | 2 (API) / 1 (bridge) | Must have | `[NEW]` | UC-2.1: Wire handler for item data changes · UC-2.2: Trigger item updates via panel bridge action |
| FR-003 | Expose click and keyboard events on Repeater item elements with automatic item context | 1 | Critical | `[NEW]` | UC-3.1: Wire click handler scoped to the clicked item · UC-3.2: Wire keyboard event scoped to the active item |
| FR-004 | Expose hover and mouse events on Repeater item elements with automatic item context | 1 | Must have | `[NEW]` | UC-4.1: Wire hover handler scoped to the hovered item |
| FR-005 | Ensure hover and mouse event delivery works on dataset-connected Repeaters | 1 (platform dependency) | Must have | `[NEW]` | UC-5.1: Hover effects work identically on static and dataset-connected Repeaters |

---

## Functional Requirements

---

<a name="intent-1"></a>
### Intent 1 — Wire Repeater lifecycle events from the Studio 2 Inspector

---

<a name="fr-001"></a>
#### FR-001 `[MODIFY]` — Surface onItemReady and onItemRemoved as event handler properties in the Inspector

| Field | Value |
|-------|-------|
| **ReqID** | FR-001 |
| **Business Title** | Wire Repeater lifecycle events from the Inspector |
| **Description** | When a Repeater is selected in the Studio 2 Inspector, developers can see and attach function-bound event handlers for `onItemReady` and `onItemRemoved` directly — without navigating to the Code panel. Both events are exposed as named event handler properties at the Repeater level. |
| **Phase** | 1 |
| **Priority** | Critical |
| **Delta from current implementation** | `[MODIFY]` |
| **Business Rationale** | Developers and partners today must leave the Inspector and open a separate Code panel to discover lifecycle event handlers for the Repeater. Many never find them, even though the underlying capability is production-ready. Moving these into the Inspector removes the discovery barrier and aligns with Studio 2's goal of making event handlers first-class in the visual editor. |
| **Technical Rationale** | Per internal-discovery "Existing Capabilities," `onItemReady` and `onItemRemoved` are implemented in the Velo `$w` Repeater API and are already wirable from the Properties & Events panel in the Code workspace. No new API contract is required. Studio 2 must expose these as named event handler properties in the Inspector, backed by the function binding model. |
| **Change vs. existing** | **Stays the same:** Velo API contracts for `onItemReady` and `onItemRemoved`. **Changes:** The surface — both events are exposed as event handler properties in the Studio 2 Inspector when a Repeater is selected; logic is attached via function binding (not raw code stubs). |

---

**Use Case UC-1.1 — Attach a handler that runs when new Repeater items are rendered**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer wires a function to run each time a new Repeater item is rendered, with that item's data and context automatically available |
| **Use Case Description** | A developer building a product card grid wants to run initialization logic the moment each item is ready — for example, controlling a badge's visibility based on item data. Today this requires writing Velo code in the Code panel. With FR-001, the developer selects the Repeater in the Inspector, finds the "When item is ready" event handler property, and binds a function. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects a Repeater on the canvas and opens the Inspector |
| **Intent Binding** | Intent 1, sub-intent 1a |
| **Desired feelings** | Capable, in control — "I can set up this Repeater's behavior right here" |

**Happy Flow:**
1. Developer selects a Repeater on the canvas.
2. The Inspector shows the Repeater's event handler properties, including "When item is ready" (`onItemReady`).
3. Developer selects the property and binds a function via the function binding control.
4. System confirms the binding; the bound function will fire for each Repeater item when it is created, receiving the item's data and item context automatically.
5. Developer previews; the bound function runs for each item as items load.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-1.1.1 | Error State | Critical | Bound function throws a runtime error during item rendering | Error is captured and reported per item with item ID and function name; remaining items continue rendering unaffected |
| EC-1.1.2 | Empty State | Important | Repeater has no items (dataset returns empty or data array is empty) | `onItemReady` fires zero times; bound function is never called; no error is thrown |
| EC-1.1.3 | Loading State | Important | Dataset is still loading when the Inspector event property is rendered | Event handler property is shown and bindable regardless of data load state; system does not block wiring during load |
| EC-1.1.4 | Validation & Input Constraints | Important | Developer attempts to bind a function that does not exist in the current page scope | System prevents the binding from being saved and surfaces a clear error identifying the unresolvable function name |
| EC-1.1.5 | Permission & Access Control | Nice-to-Have | Developer account does not have Velo/code access | Event handler properties are visible in the Inspector but the binding control is disabled with a clear explanation of the access requirement |
| EC-1.1.6 | Concurrent & State Changes | Important | Repeater's data is updated while items are already rendered | `onItemReady` fires only for newly created items; existing items with unchanged IDs do not trigger a new `onItemReady` call |
| EC-1.1.7 | Incomplete & Partial States | Important | Developer begins function binding but does not complete it | Event handler property returns to unbound state; no partial binding is saved |
| EC-1.1.8 | Boundary Conditions | Nice-to-Have | Repeater renders a large number of items (500+) | `onItemReady` fires for each item; system imposes no execution cap on the bound function |

---

**Use Case UC-1.2 — Attach a handler that runs when a Repeater item is removed**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer wires a function to run when a Repeater item is removed by a data update, with the removed item's data available |
| **Use Case Description** | A developer building a shopping cart Repeater wants to update a running total whenever an item is removed from the dataset. With FR-001, the developer wires an `onItemRemoved` handler directly from the Inspector. The system also communicates that this event does not fire for statically defined editor items — only for data-driven removals. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects a Repeater on the canvas and opens the Inspector |
| **Intent Binding** | Intent 1, sub-intent 1b |
| **Desired feelings** | Informed, confident — "I know exactly when this will fire and when it won't" |

**Happy Flow:**
1. Developer selects a Repeater on the canvas.
2. The Inspector shows "When item is removed" (`onItemRemoved`) as an event handler property.
3. Developer binds a function; the system surfaces a contextual note that `onItemRemoved` fires only for items removed by data updates — not for statically defined items removed from the editor.
4. System confirms the binding.
5. Developer previews; when the connected dataset updates and an item is removed, the bound function fires with that item's data.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-1.2.1 | Boundary Conditions | Critical | `onItemRemoved` is wired on a Repeater with only statically defined (editor-defined) items | `onItemRemoved` does not fire; system surfaces an in-context note at bind time that this event applies only to data-update removals, not to static items |
| EC-1.2.2 | Error State | Critical | Bound function throws during item removal | Error is captured per item; remaining removals in a bulk operation are not affected |
| EC-1.2.3 | Concurrent & State Changes | Important | Multiple items are removed simultaneously by a bulk data update | `onItemRemoved` fires once per removed item; calls are sequential |
| EC-1.2.4 | Empty State | Important | All items are removed (dataset returns empty array) | `onItemRemoved` fires for each previously existing item; Repeater ends in empty state with no further calls |
| EC-1.2.5 | Incomplete & Partial States | Important | A data update partially removes items (some items removed, some not) | `onItemRemoved` fires only for items that were actually removed; items that remain do not trigger the event |
| EC-1.2.6 | Loading State | Nice-to-Have | Dataset is loading during initial render | `onItemRemoved` does not fire during initial load; fires only in response to subsequent data updates |
| EC-1.2.7 | Validation & Input Constraints | Important | Same as EC-1.1.4 | Same behavior as EC-1.1.4 |
| EC-1.2.8 | Permission & Access Control | Nice-to-Have | Same as EC-1.1.5 | Same behavior as EC-1.1.5 |

---

**Codebase Implications — FR-001:**
- **API:** `onItemReady($item, itemData, index)` and `onItemRemoved($item, itemData, index)` exist in the Velo `$w` Repeater API (confirmed: [dev.wix.com/docs/velo/velo-only-apis/$w/repeater](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater)). No new API contract is needed.
- **Impacted modules:** Studio 2 Inspector panel (Repeater component section); function binding control; event handler property registry for the Repeater component type.
- **Data/storage:** Function bindings (event property → page-scoped function name) must be stored as part of the page's component configuration — same storage model as other function-bound properties in Studio 2.
- **Permission implications:** Requires Velo/code access. The Inspector must respect this permission boundary — show but disable binding for accounts without Velo access.
- **PII implications:** No PII is transmitted via event handler bindings or lifecycle event payloads. `itemData` may contain CMS collection field values — binding layer must not log or transmit `itemData` contents.
- **Observability:** Bind/unbind actions logged with timestamp, element ID, event type, user ID. Runtime bound-function errors must surface in the site's developer error log with item ID, event type, function name.
- **Testing:** Verify `onItemReady` fires for each item on data array assignment; verify `onItemRemoved` fires for dataset-removed items but NOT for editor-removed static items; verify bound function receives correct `itemData` and item context.
- **Technical risk:** Sept 2024 `$item` selector regression — if the function binding framework passes item context directly (not via `$item`), this regression is bypassed for all developers wiring via the Inspector. Confirm passing model with platform team before shipping.
- **Dependencies:** Function binding infrastructure (separate team/spec).

---

<a name="fr-002"></a>
#### FR-002 `[NEW]` — Add onItemUpdated as a Repeater lifecycle event property

| Field | Value |
|-------|-------|
| **ReqID** | FR-002 |
| **Business Title** | Get notified when a Repeater item's data changes |
| **Description** | When an existing Repeater item's data is updated in place (via `.data` assignment), a lifecycle event fires automatically — closing the gap that today forces developers to write manual `forEachItem()` boilerplate after every data update. A bridge action (backed by `forItems()`) is exposed in Phase 1 while the platform API is pending. |
| **Phase** | Phase 2 for `onItemUpdated` event runtime · Phase 1 for Inspector surface design and `forItems()` bridge |
| **Priority** | Must have |
| **Delta from current implementation** | `[NEW]` |
| **Business Rationale** | Missing `onItemUpdated` is the #1 developer pain point in user voice research. Every developer building a dynamic Repeater — live score tables, filtered product catalogs, real-time dashboards — writes the same manual boilerplate after every data update because no lifecycle event fires. Shipping `onItemUpdated` makes Studio 2 uniquely capable; no competitor exposes this in a visual editor. |
| **Technical Rationale** | Per internal-discovery: "`onItemUpdated` does not exist in the Velo API. Setting `.data` on an existing Repeater item does not trigger any callback." Adding the event requires the Velo/platform team to detect when an item's data object changes (by ID comparison) and fire a new callback. This is a net-new API — not a UI-only change. The Inspector property slot and `forItems()` bridge can be delivered in Phase 1; the runtime event delivery is Phase 2. |
| **Change vs. existing** | Net-new. No current lifecycle event fires on item data update. Requires: (1) New Velo API: `onItemUpdated($item, itemData, previousItemData, index)` — Platform team work; (2) Inspector exposes the property with a clear Phase 2 availability indicator; (3) Runtime fires the event when an item with an existing ID receives changed data. Bridge: `forItems()` exposed as a panel-level "Update selected items" action in Phase 1. |

---

**Use Case UC-2.1 — Wire a handler that fires automatically when item data changes**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer is notified when an existing Repeater item's data changes, without writing manual iteration boilerplate |
| **Use Case Description** | A developer building a live sports scores Repeater wants to animate a score-change highlight whenever a score updates. Today this requires calling `forEachItem()` manually after each data assignment. With `onItemUpdated`, the system calls the bound function automatically when any item's data changes in place, passing both the new and previous item data. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects a Repeater in the Inspector; `onItemUpdated` Inspector property is visible (Phase 1 with availability indicator; Phase 2 active) |
| **Intent Binding** | Intent 1, sub-intent 1c |
| **Desired feelings** | Relieved, empowered — "Finally, no more boilerplate" |

**Happy Flow:**
1. Developer selects a Repeater on the canvas.
2. Inspector shows "When item data changes" (`onItemUpdated`) as an event handler property. In Phase 1, the property is visible with a clear indication that it requires a platform update (and an expected timeline if known). In Phase 2, the property is active.
3. Developer binds a function (Phase 2 active state).
4. System confirms the binding; the bound function will fire when any existing item's data object changes, receiving new data, previous data, and item context.
5. Developer previews; when the site updates an item's data, the bound function fires automatically with that item's updated and previous data.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-2.1.1 | Validation & Input Constraints | Critical | Updated data object is identical to the previous (no actual change) | `onItemUpdated` does NOT fire; system compares item data by value, not reference, to prevent spurious triggers |
| EC-2.1.2 | Error State | Critical | Bound function throws during an item data update | Error is captured per item; other item updates in the same batch are not affected |
| EC-2.1.3 | Incomplete & Partial States | Important | Inspector property is visible in Phase 1 before the platform API is deployed | Property is shown with a clear "unavailable — coming in a future release" indicator; binding is disabled; developer is not able to save a binding that will never fire |
| EC-2.1.4 | Boundary Conditions | Important | Bulk data update changes many items simultaneously | `onItemUpdated` fires once per changed item; items whose data is unchanged (same ID, same value) are skipped |
| EC-2.1.5 | Concurrent & State Changes | Important | A second data update arrives while the first `onItemUpdated` handler is still executing | Second update queues; handlers are called sequentially per item; no data race |
| EC-2.1.6 | Empty State | Important | `onItemUpdated` is wired but the Repeater has no items | No calls are made; no error is thrown |
| EC-2.1.7 | Loading State | Nice-to-Have | Dataset is loading initial data | `onItemUpdated` does not fire for the initial load — initial item creation is covered by `onItemReady` |
| EC-2.1.8 | Permission & Access Control | Nice-to-Have | Same as EC-1.1.5 | Same behavior as EC-1.1.5 |

---

**Use Case UC-2.2 — Trigger item-specific updates via a panel bridge action (Phase 1 interim)**

| Field | Value |
|-------|-------|
| **Use Case Goal** | While `onItemUpdated` is pending platform work, developer can trigger item-specific update logic from the panel without writing `forEachItem()` boilerplate |
| **Use Case Description** | A developer needs to apply a price-formatting function to specific items after a data refresh. As a bridge, the Inspector exposes a "Update selected items" panel action backed by `forItems()`, allowing the developer to bind a function and specify the items to update — without writing manual iteration code. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects a Repeater and opens the Inspector; `onItemUpdated` is not yet available (Phase 1) |
| **Intent Binding** | Intent 1, sub-intent 1c (bridge) |
| **Desired feelings** | Unblocked — "I can move forward without waiting for the platform" |

**Happy Flow:**
1. Developer selects the Repeater in the Inspector.
2. Developer finds the "Update selected items" panel action (backed by `forItems()`).
3. Developer binds a function and specifies a target item ID or all items.
4. At runtime: when triggered (e.g., from another event), `forItems()` is called with the specified IDs and the bound function runs per item.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-2.2.1 | Error State | Important | One or more specified item IDs do not exist in the current Repeater | `forItems()` silently skips non-existent IDs; bound function is called only for valid IDs; a developer-facing warning is surfaced listing skipped IDs |
| EC-2.2.2 | Boundary Conditions | Important | All current item IDs are passed | Behaves equivalently to `forEachItem()` — processes all items |
| EC-2.2.3 | Empty State | Important | Repeater has no items when the action is triggered | `forItems()` runs with an empty set; bound function is not called; no error |
| EC-2.2.4 | Concurrent & State Changes | Important | Data update changes item IDs while the action is executing | Action operates on the snapshot of IDs at execution time; new/removed items since execution start are not included |
| EC-2.2.5 | Incomplete & Partial States | Important | Developer specifies an item selector expression that resolves to zero items | Same behavior as EC-2.2.3 |
| EC-2.2.6 | Validation & Input Constraints | Important | Same as EC-1.1.4 | Same behavior as EC-1.1.4 |
| EC-2.2.7 | Loading State | Nice-to-Have | Data has not yet loaded when the action is triggered | Action operates on currently loaded items only |
| EC-2.2.8 | Permission & Access Control | Nice-to-Have | Same as EC-1.1.5 | Same behavior as EC-1.1.5 |

---

**Codebase Implications — FR-002:**
- **Phase 2 API (`onItemUpdated`):** Net-new Velo API event. The platform team must detect when an item's data object changes by comparing item data keyed by item ID on each `.data` assignment. Proposed signature: `onItemUpdated($item, itemData, previousItemData, index)`. Final API contract to be aligned with the Velo/platform team — this spec defines the requirement; the Velo team owns the contract design.
- **Phase 1 surface:** Inspector exposes the `onItemUpdated` property slot with a clear "coming in a future release" state. Binding must be disabled in Phase 1 to prevent developers from wiring a handler that can never fire.
- **Phase 1 bridge (`forItems()`):** `forItems([ids], $item, itemData, index)` exists in the Velo API. The Inspector needs a panel action that invokes it with function-bound per-item logic. This is a UI-only change — no new API required.
- **Impacted modules:** Studio 2 Inspector (Repeater event handler properties, phase indicator component); Velo runtime (Phase 2 — data change detection); function binding control.
- **Data/storage:** Same as FR-001. `previousItemData` in the Phase 2 callback requires the runtime to retain a snapshot of item data between assignments — storage design decision for the Velo team.
- **Permission implications:** Same as FR-001.
- **PII implications:** `itemData` and `previousItemData` may contain CMS field values — same handling as FR-001.
- **Dependencies:** Velo/platform team commitment for Phase 2 `onItemUpdated` runtime. Function binding infrastructure for both phases.
- **Technical risk (from internal-discovery):** "`onItemUpdated` requires new platform API — not a UI-only change." Must be flagged as a blocking dependency for Phase 2 delivery.

---

<a name="intent-2"></a>
### Intent 2 — Wire item-level events with automatic per-item scoping

---

<a name="fr-003"></a>
#### FR-003 `[NEW]` — Expose click and keyboard events on Repeater item elements with automatic item context

| Field | Value |
|-------|-------|
| **ReqID** | FR-003 |
| **Business Title** | Wire click and keyboard events per Repeater item from the Inspector |
| **Description** | Elements inside a Repeater item (e.g., buttons, images, text) can be selected and have click and keyboard event handlers attached as properties in the Inspector. The bound function automatically receives the interacted item's data and context — without any manual selector workaround. |
| **Phase** | 1 |
| **Priority** | Critical |
| **Delta from current implementation** | `[NEW]` |
| **Business Rationale** | Today, developers must write Velo code in the Code panel to wire click/keyboard events on Repeater item elements, and they must know the non-obvious `$w.at(event.context)` workaround to get the correct item's data. Wrong-item bugs are common and hard to debug. Studio 2's model — where selecting an element inside a Repeater item exposes item-level event handler properties in the Inspector, with item context passed automatically — eliminates both the discovery friction and the entire scoping bug class. |
| **Technical Rationale** | Per internal-discovery "How It Works Today": child elements in a Repeater template support standard events (click, hover, focus) in the Properties & Events panel, with item context accessible via `event.context.itemId`. The events exist in the Velo API. With the function binding model, the Inspector passes item context directly to the bound function — the developer never touches `$w` or `$item` selectors, so the Sept 2024 `$item` regression is bypassed at the surface level for all Inspector-wired handlers. |
| **Change vs. existing** | Net-new surface. Events exist in the Velo API for child elements; what's new is: exposing them as item-level event handler properties in the Inspector when an element inside a Repeater item is selected; and the function binding model passing item context directly (not via `$item` or `$w.at(event.context)`). |

---

**Use Case UC-3.1 — Wire a click handler scoped to the clicked Repeater item**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer attaches a click handler to an element inside a Repeater item; the bound function automatically receives the specific clicked item's data |
| **Use Case Description** | A developer building a favorites list Repeater wants each item's "Remove" element to trigger logic using that item's data — not data from the first or all items. With FR-003, the developer selects the element inside the Repeater item in the Inspector, finds the "When clicked" event handler property, and binds a function. The system ensures the bound function always receives the correct item's data when that specific item is clicked. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects an element inside a Repeater item on the canvas |
| **Intent Binding** | Intent 2, sub-intent 2a |
| **Desired feelings** | Correct by default — "I clicked item #3 and item #3's data came through; no debugging required" |

**Happy Flow:**
1. Developer selects an element inside a Repeater item on the canvas.
2. Inspector recognizes the element is inside a Repeater item and shows event handler properties, including "When clicked" (`onClick`).
3. Developer binds a function via the function binding control.
4. System confirms the binding; the bound function will receive the interacted item's data and item context automatically on each click.
5. Developer previews; clicking the element on item #3 triggers the bound function with item #3's data — not item #1's.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-3.1.1 | Concurrent & State Changes | Critical | User clicks elements in multiple items rapidly before the first handler completes | Each click fires the bound function independently with the correct item's context; no item context bleed between concurrent invocations |
| EC-3.1.2 | Error State | Critical | Bound function throws at runtime | Error is captured with item ID and element ID; other items are not affected |
| EC-3.1.3 | Boundary Conditions | Important | Element is inside a nested Repeater (Repeater inside a Repeater item) | Item context is scoped to the innermost Repeater item; flagged as a dependency on the nested Repeater spec |
| EC-3.1.4 | Validation & Input Constraints | Important | No function is bound; element is clicked during preview | No error; click fires but executes a graceful no-op |
| EC-3.1.5 | Incomplete & Partial States | Important | Developer begins function binding but does not complete it | No partial binding saved; event property returns to unbound state |
| EC-3.1.6 | Permission & Access Control | Important | Developer account does not have Velo/code access | Event handler properties are visible in Inspector but binding control is disabled |
| EC-3.1.7 | Empty State | Nice-to-Have | Repeater has no items | No click events fire; no error |
| EC-3.1.8 | Loading State | Nice-to-Have | Dataset is loading when element is clicked | Click events do not fire during initial render; only fire on fully rendered items |

---

**Use Case UC-3.2 — Wire a keyboard event scoped to the active Repeater item**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer attaches a keyboard event handler to a keyboard-interactive element inside a Repeater item; the bound function receives the specific active item's data |
| **Use Case Description** | A developer building a Repeater-based editable list wants to capture the Enter key on each item's text input and process that item's data. With FR-003, the developer wires the keyboard event handler from the Inspector, and the bound function receives only the data for the item whose input was used. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects a keyboard-interactive element (e.g., input field) inside a Repeater item |
| **Intent Binding** | Intent 2, sub-intent 2a |
| **Desired feelings** | Confident — "The event fired on the item I typed in" |

**Happy Flow:** Same structure as UC-3.1 — select, bind, confirm; at runtime the bound function receives the specific item's data based on which item's element received keyboard input.

**Edge Cases:** Same as UC-3.1, plus:

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-3.2.1 | Validation & Input Constraints | Important | Developer wires a keyboard event on a non-focusable element inside the item | Inspector does not offer keyboard event handler properties for non-focusable elements; or if offered, surfaces a clear warning that the element must be keyboard-reachable for the event to fire |

---

**Codebase Implications — FR-003:**
- **API:** `onClick`, `onKeyPress`, and related child-element events exist in the Velo `$w` API for standard elements. Per internal-discovery: item context is accessible via `event.context.itemId`. The function binding layer extracts item context from `event.context` and passes it directly to the bound function — no `$w.at(event.context)` workaround needed at the Inspector layer.
- **Impacted modules:** Studio 2 Inspector (element-level event handler properties, triggered when selected element is inside a Repeater item); function binding control; Inspector must have Repeater-ancestry awareness to know when an element is inside a Repeater and to surface item-scoped properties automatically.
- **Architectural dependency:** The Inspector must be able to determine whether a selected element is inside a Repeater item (Repeater-ancestry check). If this capability is not available in the Inspector architecture, it is a blocking prerequisite — must be confirmed with the editor infrastructure team.
- **Data/storage:** Same as FR-001.
- **Permission implications:** Same as FR-001.
- **PII implications:** `itemData` passed to bound functions may contain CMS field values; same handling as FR-001.
- **Observability:** Per-item event firing should be loggable with item ID and element ID for debugging.

---

<a name="fr-004"></a>
#### FR-004 `[NEW]` — Expose hover and mouse events on Repeater item elements with automatic item context

| Field | Value |
|-------|-------|
| **ReqID** | FR-004 |
| **Business Title** | Wire hover and mouse events per Repeater item from the Inspector |
| **Description** | Elements inside a Repeater item can have hover and mouse event handlers (`onMouseIn`, `onMouseOut`) attached as properties in the Inspector. The bound function automatically receives the hovered item's data and context. Event delivery on dataset-connected Repeaters depends on the platform fix in FR-005. |
| **Phase** | 1 (surface; delivery on dataset-connected Repeaters gated on FR-005) |
| **Priority** | Must have |
| **Delta from current implementation** | `[NEW]` |
| **Business Rationale** | Hover effects on Repeater items — overlays on portfolio cards, hover states on product grids — are one of the most common Partner use cases. The silent failure on dataset-connected Repeaters makes this capability unreachable for the majority of real-world configurations. Exposing hover as a first-class item-level event property, with a clear dependency indicator for dataset-connected Repeaters, unblocks this pattern and aligns with the Studio 2 goal of reliable, predictable interactions. |
| **Technical Rationale** | `onMouseIn` and `onMouseOut` exist in the Velo API for child elements. The Inspector surface change is the same model as FR-003. The delivery failure on dataset-connected Repeaters is a separate platform-level bug (FR-005) — the surface can be shipped independently. The Inspector should surface a dependency note when hover events are wired on a dataset-connected Repeater. |
| **Change vs. existing** | Net-new surface for hover/mouse at the item level in the Inspector. Delivery on dataset-connected Repeaters is blocked on FR-005. |

---

**Use Case UC-4.1 — Wire hover handlers scoped to the hovered Repeater item**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer attaches `onMouseIn`/`onMouseOut` handlers to an element inside a Repeater item; each event fires with the hovered item's data — not all items' |
| **Use Case Description** | A developer building a portfolio Repeater wants each project card to show a title overlay on hover. The overlay must show the hovered project's title specifically. With FR-004, the developer selects the card element, wires `onMouseIn` and `onMouseOut` from the Inspector, and the system handles item scoping automatically. For dataset-connected Repeaters, the Inspector surfaces a dependency note pointing to FR-005. |
| **Actor** | Developer or Partner |
| **Trigger** | Developer selects an element inside a Repeater item on the canvas |
| **Intent Binding** | Intent 2, sub-intent 2b |
| **Desired feelings** | Reliable — "Hovering item #2 shows item #2's data, every time" |

**Happy Flow:**
1. Developer selects an element inside a Repeater item on the canvas.
2. Inspector shows event handler properties including "When mouse enters" (`onMouseIn`) and "When mouse leaves" (`onMouseOut`).
3. Developer binds a function for `onMouseIn`; the bound function will receive the hovered item's data and context automatically.
4. Developer optionally binds a function for `onMouseOut` (independent of `onMouseIn` binding).
5. If the Repeater is dataset-connected, the Inspector surfaces a contextual note that event delivery depends on the platform fix in FR-005.
6. Developer previews (on a non-dataset Repeater or after FR-005 is resolved); hovering item #2 fires `onMouseIn` with item #2's data; leaving fires `onMouseOut` with item #2's data.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-4.1.1 | Concurrent & State Changes | Critical | User moves mouse rapidly across multiple items | `onMouseIn` and `onMouseOut` fire independently per item with the correct item context; no context bleed between overlapping hover events |
| EC-4.1.2 | Boundary Conditions | Critical | Repeater is dataset-connected and FR-005 is not yet resolved | Inspector surface is correct; event delivery may not work; system surfaces a dependency note at bind time; does not block the developer from wiring the handler |
| EC-4.1.3 | Error State | Critical | Bound function throws at runtime | Error captured per item with item ID; other items continue unaffected |
| EC-4.1.4 | Incomplete & Partial States | Important | Only `onMouseIn` is wired; `onMouseOut` is not | System allows asymmetric binding; `onMouseIn` fires normally; absence of `onMouseOut` handler is not an error |
| EC-4.1.5 | Loading State | Important | Items are rendering; user attempts to hover during render | Hover events do not fire until item is fully rendered |
| EC-4.1.6 | Validation & Input Constraints | Important | Same as EC-3.1.4 | Same behavior |
| EC-4.1.7 | Permission & Access Control | Nice-to-Have | Same as EC-3.1.6 | Same behavior |
| EC-4.1.8 | Empty State | Nice-to-Have | No items in Repeater | No hover events fire; no error |

---

**Codebase Implications — FR-004:**
- **API:** `onMouseIn` and `onMouseOut` exist in the Velo `$w` API for child elements. Same architecture as FR-003 — function binding passes item context directly.
- **Dataset-connected dependency note:** Inspector must detect whether the Repeater ancestor is dataset-connected and, if so, surface an inline note linking to the platform dependency (FR-005). This requires the Inspector to know the Repeater's data binding state.
- **Impacted modules:** Same as FR-003, plus the dataset-connection state reader in the Inspector.
- **Permission/PII implications:** Same as FR-003.
- **Dependencies:** FR-005 (platform bug fix) for full delivery on dataset-connected Repeaters. FR-003 architectural pattern (Repeater-ancestry awareness) is a prerequisite.

---

<a name="intent-3"></a>
### Intent 3 — Hover and mouse events work reliably on dataset-connected Repeaters

---

<a name="fr-005"></a>
#### FR-005 `[NEW]` — Ensure hover and mouse event delivery works on dataset-connected Repeaters

| Field | Value |
|-------|-------|
| **ReqID** | FR-005 |
| **Business Title** | Fix silent hover/mouse event failure on dataset-connected Repeaters |
| **Description** | Hover and mouse events (`onMouseIn`, `onMouseOut`) must fire identically whether a Repeater is populated statically or connected to a CMS dataset. This FR defines the acceptance contract for the platform-level bug fix. The root cause is unconfirmed — it may be in the dataset layer or the Repeater event system. |
| **Phase** | 1 (platform fix; Inspector surface for FR-004 is independent and ships without this) |
| **Priority** | Must have |
| **Delta from current implementation** | `[NEW]` |
| **Business Rationale** | Dataset-connected Repeaters are the most common production configuration. Partners building product grids, portfolios, and CMS-driven galleries all use dataset connections. When hover events silently fail in this configuration, partners must either remove the dataset connection (losing CMS functionality) or remove the hover effect (degrading UX). No competitor has this limitation — Webflow, Framer, and Bubble all decouple data binding from event delivery. This is a unique competitive disadvantage that Studio 2 must resolve. |
| **Technical Rationale** | Per internal-discovery Technical Risk: "Hover/mouse events break silently on dataset-connected Repeaters. Root cause unknown — may be dataset layer or Repeater event system. Root cause investigation needed before Studio 2 ships." This FR defines the expected contract; the Velo/platform or CMS/Dataset team must investigate and fix the root cause. The fix must be confirmed and regression-tested before FR-004's hover delivery on dataset-connected Repeaters is declared complete. |
| **Change vs. existing** | Net-new fix. Root cause must first be identified via a platform investigation. The fix decouples dataset connection state from mouse event delivery in the Repeater event pipeline. |

---

**Use Case UC-5.1 — Hover effects work identically on static and dataset-connected Repeaters**

| Field | Value |
|-------|-------|
| **Use Case Goal** | Developer's `onMouseIn`/`onMouseOut` handlers on Repeater item elements fire reliably whether the Repeater is static or dataset-connected |
| **Use Case Description** | A Partner builds a portfolio site with project cards in a dataset-connected Repeater. Each card should show a title overlay on hover. Today `onMouseIn`/`onMouseOut` work in isolation but silently stop when the dataset is connected. FR-005 requires this to be fixed: hover event delivery must be identical in both configurations. |
| **Actor** | Developer or Partner (authoring); end user (triggering hover at runtime) |
| **Trigger** | End user hovers over an item in a dataset-connected Repeater on the live site; developer has wired `onMouseIn`/`onMouseOut` via FR-004 |
| **Intent Binding** | Intent 3, sub-intent 3a |
| **Desired feelings** | Predictable, reliable — "It just works, regardless of where the data comes from" |

**Happy Flow:**
1. Repeater is connected to a CMS dataset; items are populated from the dataset.
2. Developer has wired `onMouseIn` and `onMouseOut` event handlers on item elements via FR-004.
3. End user hovers over item #2 on the live site.
4. `onMouseIn` fires for item #2 with item #2's data; the bound function executes normally.
5. End user moves mouse away; `onMouseOut` fires for item #2.
6. Behavior is identical to a static (non-dataset-connected) Repeater.

**Edge Cases:**

| # | Category | Severity | Trigger | Expected System Behavior |
|---|----------|----------|---------|--------------------------|
| EC-5.1.1 | Concurrent & State Changes | Critical | Dataset refreshes while user is hovering over an item (item is re-rendered) | `onMouseOut` fires for the interrupted item if hover state is lost during re-render; `onMouseIn` fires when user re-enters the re-rendered item |
| EC-5.1.2 | Incomplete & Partial States | Critical | Platform fix resolves `onMouseIn` delivery but not `onMouseOut` (or vice versa) | Both `onMouseIn` and `onMouseOut` must be verified as fully functional before the FR-005 dependency is marked resolved |
| EC-5.1.3 | Boundary Conditions | Critical | Developer switches a working static Repeater to dataset-connected | Hover events must continue working after the dataset is connected — no behavior regression |
| EC-5.1.4 | Error State | Important | Root cause is in the dataset layer and fix requires decoupling dataset updates from event suppression | Fix must not suppress mouse events during or after dataset operations; event pipeline and dataset update pipeline must be independent |
| EC-5.1.5 | Loading State | Important | Dataset is paginating or refreshing while user hovers | Hover events resume normally after items re-render; in-progress hover that was interrupted resumes with the correct item |
| EC-5.1.6 | Empty State | Important | Dataset returns empty — no items in Repeater | No hover events fire; no error |
| EC-5.1.7 | Validation & Input Constraints | Important | Root cause investigation reveals the failure is intentional behavior (not a bug) | Studio 2 PM escalates to product leadership; FR-005 scope is re-evaluated; FR-004 must document the limitation prominently |
| EC-5.1.8 | Boundary Conditions | Nice-to-Have | Repeater is connected to multiple datasets simultaneously | Hover event delivery must be consistent regardless of the number of connected datasets |

---

**Codebase Implications — FR-005:**
- **Platform dependency:** This FR is owned by the Velo/platform team or the CMS/Dataset team — root cause determination required before any code change. Studio 2 PM owns the requirement definition; platform team owns the fix.
- **Root cause investigation required:** Per internal-discovery Technical Risk: "Root cause investigation needed (dataset layer vs. Repeater event system)." Investigation must confirm whether the dataset connection is actively suppressing mouse events, or whether the Repeater event system is failing to wire child events when a dataset is connected.
- **Fix acceptance criteria:** After the fix is deployed, all edge cases in UC-5.1 must pass automated regression tests. NFR-06 (≥99.9% event delivery parity) is the quantitative acceptance threshold.
- **Studio 2 Inspector dependency:** FR-004 Inspector surface ships independently of FR-005. The Inspector dependency note (EC-4.1.2) must be removed when FR-005 is confirmed resolved.
- **Observability:** Post-fix, hover event delivery on dataset-connected Repeaters should be monitored via event pipeline instrumentation. Any suppression events should be logged as errors.

---

## Non-Functional Requirements

### Intent-Tied NFRs

| NFR ID | Category | Description | Threshold | Impl Status | Scope | Applies to FRs |
|--------|----------|-------------|-----------|-------------|-------|----------------|
| NFR-01 | Usability | Lifecycle event handler properties are visible in the Inspector by default when a Repeater is selected — no Code panel navigation or secondary click required | Developer can locate and wire a lifecycle event handler (onItemReady or onItemRemoved) within 60 seconds of selecting a Repeater, with no prior knowledge of the Code panel | `[NEW]` | Delta-only | FR-001, FR-002 |
| NFR-02 | Error Messages | The `onItemRemoved` static-item exception is communicated in-context at bind time — no separate documentation lookup required | When `onItemRemoved` is bound, a contextual note about the static-item exception is surfaced within the binding experience before the developer confirms the binding | `[NEW]` | Delta-only | FR-001 |
| NFR-03 | Usability | The Inspector distinguishes between "event available now" and "event pending platform release" for `onItemUpdated`, preventing developers from wiring a handler that can never fire | `onItemUpdated` Inspector property clearly indicates Phase 1 (unavailable) vs. Phase 2 (active) state with a reason and, where known, an expected timeline; binding is disabled in Phase 1 | `[NEW]` | Delta-only | FR-002 |
| NFR-04 | Usability | Item context is automatically correct for 100% of event handlers wired via the Inspector from within a Repeater item — zero manual workaround required | Zero developer reports of wrong-item data received through function-bound handlers wired via the Inspector | `[NEW]` | Delta-only | FR-003, FR-004 |
| NFR-05 | Error Messages | When hover/mouse events are wired on a dataset-connected Repeater, the system surfaces an in-context note about the platform dependency (FR-005) — no silent failure | A dependency note is surfaced within the binding experience for `onMouseIn`/`onMouseOut` on dataset-connected Repeaters before the developer confirms the binding; note is removed once FR-005 is resolved | `[NEW]` | Delta-only | FR-004 |
| NFR-06 | Reliability | Hover and mouse event delivery on dataset-connected Repeaters is at parity with static Repeaters | ≥99.9% event delivery rate for `onMouseIn`/`onMouseOut` on dataset-connected Repeaters, measured post FR-005 fix; zero silent event suppression in the event pipeline | `[NEW]` | Delta-only | FR-005 |

---

### Cross-Cutting NFRs

| NFR ID | Category | Description | Threshold | Impl Status | Scope | Linked Intents/Feelings |
|--------|----------|-------------|-----------|-------------|-------|------------------------|
| NFR-07 | Reliability | Lifecycle events (`onItemReady`, `onItemRemoved`) fire with consistent delivery across all Repeater configurations | ≥99.5% successful event delivery for `onItemReady` and `onItemRemoved`; measured as fired events vs. expected events per Repeater data operation | `[NEW]` | Project-wide | Intents 1, 2 — from "events that don't fire" → "events that always fire" |
| NFR-08 | Performance | Inspector event handler properties for a Repeater load without perceptible latency after element selection | ≤300ms from element selection to first render of event handler properties in the Inspector; measured at P95 | `[NEW]` | Delta-only — FR-001 through FR-004 | Intent 1 — from "capable, in control" |
| NFR-09 | Accessibility | All new Inspector controls introduced by this spec (event property list, function binding control, phase indicators, inline notes) meet WCAG 2.1 AA | All new controls are keyboard-operable; all labels and states are announced correctly by major screen readers (VoiceOver, NVDA); color contrast ≥4.5:1 for text elements | `[NEW]` | Delta-only — FR-001 through FR-004 | All intents |
| NFR-10 | Supportability | Event handler bind/unbind actions and runtime errors are observable for debugging | All bind/unbind actions logged: timestamp, element ID, event type, user ID. Runtime bound-function errors include: item ID, event type, function name, stack trace. Logs accessible in the site's developer error log | `[NEW]` | Project-wide | All intents |
| NFR-11 | Documentation & Education | Developer-facing documentation and Customer Care readiness are in place before GA | Before GA: (1) Velo API reference updated for `onItemReady`, `onItemRemoved`, and `onItemUpdated` (Phase 2); (2) Studio 2 Inspector event handler guide published; (3) Customer Care readiness brief covering the `onItemRemoved` static-item exception and the dataset-connected hover dependency | `[NEW]` | Project-wide | Intents 1, 3 — from "no signal, wrong expectations" |
| NFR-12 | Security | The function binding mechanism does not allow unintended code execution or data exposure | Function binding resolves only to named functions defined in the current page scope; wildcard or dynamic function references are rejected at bind time; item context data passed to bound functions is not accessible in global scope; binding validation is enforced server-side, not client-side only | `[NEW]` | Project-wide | All intents |
| NFR-13 | Localization | All new UI strings introduced by this spec are localized and RTL-tested before GA | All new strings (event property names, inline notes, phase indicators, error messages) submitted for localization before code freeze; RTL layout verified for the Inspector event handler section on at least one RTL locale (Hebrew or Arabic) | `[NEW]` | Delta-only — FR-001 through FR-004 | All intents |

---

## Open Dependencies

The following must be resolved before Studio 2 can ship full event handler coverage for the Repeater. All are flagged as blocking or high-priority in the internal-discovery report.

| # | Dependency | Blocking for | Owner | Status |
|---|------------|-------------|-------|--------|
| OD-01 | **Sept 2024 `$item` scoping regression** — confirm whether fixed, still live, or intentional. If still live: function binding layer must be verified to bypass it (pass item context directly, not via `$item`). | FR-001, FR-003, FR-004 code stub generation | Velo/platform team | Unconfirmed |
| OD-02 | **`onItemUpdated` platform API** — Velo team must commit to designing and shipping the new lifecycle event callback. Studio 2 can spec the Inspector surface and bridge in Phase 1; event delivery requires platform work. | FR-002 Phase 2 | Velo/platform team | Not on roadmap (unconfirmed) |
| OD-03 | **Hover/mouse bug root cause** — identify whether the failure is in the dataset layer or the Repeater event system, then fix and regression-test. | FR-005, FR-004 full delivery on dataset-connected Repeaters | Velo/platform or CMS/Dataset team | Root cause unconfirmed |
| OD-04 | **Properties & Events panel ownership in Studio 2** — confirm whether the CMS group or editor infrastructure group owns the Inspector event property surface. Determines who implements FR-001 through FR-004. | All FRs | Studio 2 PM / editor infrastructure | Unresolved |
| OD-05 | **Wix Blocks Dynamic Repeater alignment** — before finalizing the Studio 2 Repeater event contract, compare with the Blocks Dynamic Repeater event model to avoid architectural divergence across Wix Repeater variants. | FR-001, FR-002 final contract | Studio 2 PM / Wix Blocks team | Not yet reviewed |
| OD-06 | **Inspector Repeater-ancestry awareness** — confirm with the editor infrastructure team that the Inspector can detect when a selected element is inside a Repeater item. Required for FR-003 and FR-004 item-level event scoping. | FR-003, FR-004 | Editor infrastructure team | Not yet confirmed |

---

*Spec generated from: project-brief-repeater-event-handlers.md, internal-discovery-repeater-event-handlers.md, terminology-research-repeater-event-handlers.md, competitor-research-repeater-event-handlers.md, user-voice-repeater-event-handlers.md, research-summary-repeater-event-handlers.md.*
*Note: `product-current-impl-repeater-event-handlers.md` was not available. Delta tagging is based on internal-discovery "Existing Capabilities" and "How It Works Today" as a substitute. Run `ck-product-current-impl` before final engineering sign-off to verify codebase-level implications.*

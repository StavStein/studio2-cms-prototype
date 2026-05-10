# Requirements Specification — Repeater in Repeater (Studio 2 CMS)

> **Status:** Draft v2 — delta tags added to §3 Summarization Table from codebase scan of `wix-private/odeditor-packages` (HEAD `8b0a2f9`, 2026-04-26). Built from the discovery suite under `artifacts/discovery/`, the existing [`blank-repeater-prd.md`](../../blank-repeater-prd.md), and the [`product-spec-repeater-pagination.md`](product-spec-repeater-pagination.md).

## Table of Contents

- [1. Overview](#1-overview)
- [2. Intent Hierarchy](#2-intent-hierarchy)
- [3. Summarization Table](#3-summarization-table)
- [4. Functional Requirements](#4-functional-requirements)
  - [4.1 Render Parent → Child Data on the Canvas (INT-001)](#41-render-parent--child-data-on-the-canvas-int-001)
  - [4.2 Inherit Parent Context Automatically (INT-002)](#42-inherit-parent-context-automatically-int-002)
  - [4.3 Safely Replace and Disconnect Nested Bindings (INT-003)](#43-safely-replace-and-disconnect-nested-bindings-int-003)
  - [4.4 Stay Oriented at the Limits (INT-004)](#44-stay-oriented-at-the-limits-int-004)
- [5. Non-Functional Requirements](#5-non-functional-requirements)
  - [5.1 Reliability](#51-reliability)
  - [5.2 Performance](#52-performance)
  - [5.3 Security](#53-security)
  - [5.4 SEO](#54-seo)
  - [5.5 Accessibility](#55-accessibility)
  - [5.6 Localization](#56-localization)
  - [5.7 Usability](#57-usability)
  - [5.8 Error Messages](#58-error-messages)
  - [5.9 Legal Compliance](#59-legal-compliance)
  - [5.10 Documentation & Education](#510-documentation--education)
  - [5.11 Supportability](#511-supportability)
- [6. Out of Scope](#6-out-of-scope)
- [7. Open Questions](#7-open-questions)
- [8. Glossary](#8-glossary)

---

## 1. Overview

**Feature summary.** A Nested Repeater is a Repeater placed inside another Repeater's item template, whose `Items` is bound to an array or multi-reference field on the Outer Repeater's parent item. Studio 2 introduces a first-class canvas flow for it — a `This item` scope in the binding picker, drag-into-item auto-binding, depth and items-per-parent caps, and replace/disconnect cascades that match the existing blank-Repeater PRD patterns.

**Problem to solve.** Builders who organize content as parent → children (Wix Stores categories with products, Wix Restaurants menus with dishes, courses with lessons, properties with contacts) cannot render that shape natively in Studio 2 today; they fall back on multi-dataset hacks, side-by-side filtered Repeaters, or Velo / Wix Blocks code, all of which break in predictable ways the Customer Care team is already labeling internally. The runtime supports the shape via Wix Blocks `include()` / `wixData.queryReferenced()`, but the canvas binding model has not exposed it.

**Strategic positioning.** Nested repeating is **table-stakes** in this category — Webflow ships up to 3 levels with drag-and-drop, Bubble ships unbounded depth with performance footguns, Squarespace lacks it entirely. Wix Studio 2 closes a parity gap that currently shows up as switching-intent threads. The Wix Help Center already has a literal request page (*"CMS Request: Attaching a Repeater onto Another Repeater"*) — the gap is acknowledged.

**v1 scope.** Canvas-only nested binding for Studio 2, depth = 2 (Outer Repeater → Nested Repeater), items-per-parent cap = 100 (Webflow parity), reusing the existing binding-platform's scope-badge mechanism. AI-assisted flows, depth ≥ 3, mobile-specific affordances, and Tables/Galleries are out of scope.

---

## 2. Intent Hierarchy

| Intent ID | Type | Intent Statement | Priority |
|---|---|---|---|
| **INT-001** | Main | As a Studio User / Self-Creator, I want to render parent → child data on a page (e.g. categories with products, meals with dishes) so I can build content-rich sites without leaving the canvas. | Critical |
| SUB-001a | Sub | I want to bind a Nested Repeater's `Items` to a Child array / Child collection of the Parent item. | Critical |
| SUB-001b | Sub | I want available parent-item array / multi-reference fields surfaced first in the binding picker, marked with a `This item` scope. | Critical |
| SUB-001c | Sub | I want auto-binding to the first available Child array when context is inherited, the same way blank Repeaters auto-bind today. | Must have |
| **INT-002** | Main | As a Studio User, I want the Nested Repeater to inherit the Parent row context automatically so I do not have to wire datasets, references, or filters manually. | Critical |
| SUB-002a | Sub | I want context to be inherited when a Repeater is dropped inside another Repeater's item, without an explicit attach action. | Critical |
| SUB-002b | Sub | I want to clearly distinguish "inherited from parent item" vs. "directly attached" contexts in the inspector. | Must have |
| SUB-002c | Sub | I want a clear initial state when the Outer Repeater is unbound — the Nested Repeater should explain that the Outer must be connected first. | Must have |
| **INT-003** | Main | As a Studio User, I want to safely change or remove nested bindings so I can iterate on layout without losing existing work or breaking grandchild bindings unexpectedly. | Critical |
| SUB-003a | Sub | I want a warning before disconnecting the Outer Repeater's `Items` if it would cascade-disconnect inner bindings, including the Nested Repeater and its child-item bindings. | Critical |
| SUB-003b | Sub | I want a warning before replacing the Outer Repeater's source if inner bindings would be affected, with an accurate count of grandchild bindings. | Critical |
| SUB-003c | Sub | I want to disconnect or replace the Nested Repeater independently of the Outer Repeater, with clear cascade copy. | Must have |
| SUB-003d | Sub | I want a recovery path (undo / revert) immediately after a destructive change. | Must have |
| **INT-004** | Main | As a Studio User / Self-Creator, I want clear feedback when I hit nesting or data limits so I never silently fail or end up with a broken page. | Critical |
| SUB-004a | Sub | I want to see the depth limit when reached and understand why I cannot nest further. | Must have |
| SUB-004b | Sub | I want to see the items-per-parent cap when exceeded, with guidance on how to trim. | Must have |
| SUB-004c | Sub | I want clear copy when the Parent item has no usable Child arrays / Child collections. | Must have |
| SUB-004d | Sub | I want clear handling for empty data — when a Parent row's Child array is present but empty. | Must have |
| SUB-004e | Sub | I want clear visibility when the Nested Repeater's attached context differs from its `Items` source (mismatch). | Must have |

---

## 3. Summarization Table

Delta tags based on codebase scan of `wix-private/odeditor-packages` (HEAD `8b0a2f9`, 2026-04-26). Key reusable building blocks that already exist: `ComponentDataContextAPI` (`isRepeater`, `isInDynamicContainer`, `hasBindings`, `getPropertyBinding`, `unbindProperty`), `UnconfiguredRepeaterIndicator`, `BlockedRepeaterBanner`, `CmsConnectButton`, `CmsControlsWrapper`, `CmsBindingTag`, and the `CmsActionsAPI` Manage Data action. The `CMS/propertyBindingPanel` (where binding-picker scope lives) is owned by the external CMS packages (`@wix/wix-data-client-editor3-packages` etc.) loaded lazily — not in this repo.

| ReqID | Intent ID | Use Cases | Codebase Implications? | Delta | Priority | Phase |
|---|---|---|---|---|---|---|
| FR-001 | INT-001 / SUB-001a, b | UC-1: Bind Nested Repeater `Items` from Parent item; UC-2: See parent-item fields ranked first; UC-3: Surface multi-reference vs. array distinction | Extend binding-picker scope handling (in external CMS packages), reuse `include()` / `queryReferenced()` runtime primitive | **[NEW]** `This item` scope in `CMS/propertyBindingPanel`; **[NEW]** parent-row query wiring | Blocker | Phase 1 |
| FR-002 | INT-001 / SUB-001c | UC-1: Auto-bind to first parent Child array on drop; UC-2: Skip auto-bind when `Items` is already set | Extend after-add handler (`createAfterAddRepeaterWithCollectionComponentHandler`) and drop logic | **[MODIFY]** add-panel after-add handler; **[NEW]** drop-into-item auto-bind logic | Critical | Phase 1 |
| FR-003 | INT-002 / SUB-002a | UC-1: Inherit Parent row context on drop; UC-2: Re-establish inheritance when Outer's binding changes | New context propagation in `ComponentDataContextAPI` — no inherited-context concept exists today | **[NEW]** `isNestedRepeater` / `getParentRepeaterContext` on `ComponentDataContextAPI` | Blocker | Phase 1 |
| FR-004 | INT-002 / SUB-002b | UC-1: Distinguish inherited vs. directly attached contexts in the Repeater context card; UC-2: Allow direct attach on top of inheritance for the mismatch case | Context card UI doesn't exist for nested case | **[NEW]** inspector context-card render path for Nested Repeater | Critical | Phase 1 |
| FR-005 | INT-002 / SUB-002c | UC-1: Show "Outer Repeater not connected" state on Nested Repeater when Outer is unbound | `UnconfiguredRepeaterIndicator` + `BlockedRepeaterBanner` exist but only handle "no context attached" — no "outer unbound" variant. `isUnconfiguredRepeater` checks for `cmsTemplates.*` provider only; a Nested Repeater with an inherited (non-self-attached) context will incorrectly show as "unconfigured" today. | **[MODIFY]** `isUnconfiguredRepeater` to handle nested case; **[MODIFY]** `UnconfiguredRepeaterIndicator` + `BlockedRepeaterBanner` copy for "outer not connected" variant | Must have | Phase 1 |
| FR-006 | INT-003 / SUB-003a, b | UC-1: Warn on Outer disconnect with grandchild count; UC-2: Warn on Outer replace with grandchild count; UC-3: Cancel preserves all bindings | `unbindProperty` exists; cascade-warning dialog with grandchild-count computation does not | **[NEW]** cascade-warning dialog; **[NEW]** 2-level binding-impact counter | Critical | Phase 1 |
| FR-007 | INT-003 / SUB-003c | UC-1: Disconnect Nested Repeater independently; UC-2: Replace Nested Repeater's `Items` source; UC-3: Preserve layout and styling | `unbindProperty` on `ComponentDataContextAPI` is reusable; per-Repeater scoped warning copy is new | **[EXISTS]** `unbindProperty`; **[NEW]** scoped warning copy + replace flow for Nested Repeater | Critical | Phase 1 |
| FR-008 | INT-003 / SUB-003d | UC-1: Undo immediately after disconnect or replace; UC-2: Persistent undo stack across the operation | No undo-stack coverage for binding-graph operations today | **[NEW]** editor undo coverage for nested binding changes | Must have | Phase 1 |
| FR-009 | INT-004 / SUB-004a | UC-1: Block dropping a third-level Nested Repeater; UC-2: Surface depth-limit message in inspector when at depth | No depth check exists for Repeater nesting | **[NEW]** depth check on drop/paste; **[NEW]** depth-aware inspector copy | Must have | Phase 1 |
| FR-010 | INT-004 / SUB-004b | UC-1: Surface items-per-parent cap when exceeded; UC-2: Communicate that data beyond the cap is not rendered | No per-parent item cap or messaging exists | **[NEW]** runtime cap enforcement; **[NEW]** inspector cap messaging | Must have | Phase 1 |
| FR-011 | INT-004 / SUB-004c | UC-1: Communicate "no Child arrays available" when the Parent item has no list-shaped fields; UC-2: Offer recovery path | `CMS/propertyBindingPanel` has no nested empty-state for `This item` scope | **[NEW]** binding-picker empty-state for nested case (in external CMS packages) | Must have | Phase 1 |
| FR-012 | INT-004 / SUB-004d | UC-1: Render empty-Child-array state cleanly per Parent row; UC-2: Preserve layout when empty | No canvas/runtime behavior for empty inner arrays | **[NEW]** runtime empty-inner-array render; **[NEW]** layout-preservation guarantee | Must have | Phase 1 |
| FR-013 | INT-004 / SUB-004e | UC-1: Show context mismatch on Nested Repeater inspector; UC-2: Communicate which source feeds the Nested Repeater when it differs from the attached context | `CmsControlsWrapper` tag pattern is reusable; mismatch-state display is new | **[EXISTS]** `CmsControlsWrapper` / `CmsBindingTag` pattern; **[NEW]** mismatch-state indicator on Nested Repeater inspector | Must have | Phase 1 |
| NFR-001…013 | — | — | — | — | See §5 | — |

---

## 4. Functional Requirements

### 4.1 Render Parent → Child Data on the Canvas (INT-001)

#### FR-001 — Bind Nested Repeater `Items` to a Parent item's Child array

| Field | Value |
|---|---|
| **ReqID** | FR-001 |
| **Description** | Studio 2 lets a builder bind a Nested Repeater's `Items` directly to an array or multi-reference field of the Outer Repeater's Parent item, without configuring secondary datasets, filters, or code. The binding picker exposes a new `This item` scope that surfaces the Parent item's Child arrays / Child collections at the top, ranked above existing `This repeater` and page-level scopes. |
| **Change vs. existing** | Extends the existing binding picker (today: page-level contexts, `This repeater` scope) with a `This item` scope that becomes available only when the Repeater being bound is itself nested inside another Repeater's item. Multi-reference fields on the Parent item appear as selectable Child collections — this is the first place in the canvas-binding model where multi-reference fields become first-class binding sources. The runtime call is the existing `wixData.queryReferenced()` / Wix Blocks `include()` primitive. |
| **Scope** | Phase 1 |
| **Priority** | Blocker |

##### Use Cases

**UC-1 — Bind Nested Repeater `Items` from a Parent item Child array**

| Element | Value |
|---|---|
| **Use Case Goal** | The builder connects the Nested Repeater to the Parent item's Child array in one canvas action, with no manual dataset wiring. |
| **Use Case Description** | A builder is on a page that has an Outer Repeater bound to a parent collection (e.g. "Meals"). They want each rendered meal to show a sub-list of its dishes. They drop a new Repeater into the Outer Repeater's item template, click `Connect data`, and pick a Child array of the Parent item. The system binds the Nested Repeater's `Items` to that field and renders the children for each parent row. |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Render parent → child data on the canvas (Linked Intent ID: INT-001 / SUB-001a) |
| **Trigger** | Builder selects the Nested Repeater and clicks `Connect data` (action bar or canvas hat). |

**Happy Flow:**

1. Builder selects the Nested Repeater (placed inside the Outer Repeater's item template).
2. Builder initiates `Connect data`.
3. System opens the floating binding panel scoped to the Nested Repeater.
4. System surfaces the **`This item`** scope at the top of the picker. Under it, system lists the Parent item's array fields and multi-reference fields, with hierarchy labels.
5. Builder selects a Child array (or accepts the auto-selected first Child array — see FR-002).
6. Builder applies the binding.
7. System wires the Nested Repeater's `Items` to the selected Child array, scoped via the Parent row context.
8. Canvas re-renders: each Parent row's Nested Repeater now displays the resolved child rows.
9. Inspector opens on the Nested Repeater with `Properties` expanded; `Items` row shows the selected Child array with a `This item` scope badge.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|---|---|---|---|
| Error States | Critical | Runtime fails to resolve referenced rows (network or data error) | System keeps the Nested Repeater visible with placeholder content for that Parent row only; surfaces a retriable failure indicator in the inspector; other rows remain functional. |
| Empty States | Critical | Selected Child array exists but contains no entries for a given Parent row | System renders an empty Nested Repeater area for that row without collapsing the Parent row's layout (covered in detail by FR-012). |
| Loading States | Important | Initial canvas render with many parent rows that each resolve a Child collection | System batches the cross-collection fetch via `include()` / `queryReferenced()`; never issues a per-row sequential query (no N+1). |
| Validation & Input | Critical | Builder attempts to bind `Items` to a non-array, non-multi-reference field | System blocks selection; non-list fields are not selectable in the picker. |
| Boundary Conditions | Important | Parent collection has no array or multi-reference fields | System surfaces a clear empty-state message in the picker (covered by FR-011). |
| Permission & Access Control | Important | The collection backing the Child collection is permission-gated and the editor user lacks read | System surfaces a permission-error indicator and disables Apply; PII / permissions design is required. |
| Concurrent & State Changes | Important | The Outer Repeater's `Items` source is changed by another open editor session while the picker is open | System invalidates the open `This item` scope when reopened; if Apply is pressed against stale source, the binding fails gracefully and the picker reopens. |
| Incomplete & Partial States | Important | Builder closes the binding panel without applying | System leaves the Nested Repeater unbound; no partial state is persisted. |

**Desired Feelings in Output State:**

- **Empowered** — the builder did in one action what previously required three datasets and a Velo block.
- **Confident** — `This item` scope badge and clear field labels confirm what is being bound to what.

**Codebase Implications:**

| Aspect | Detail |
|---|---|
| **Impacted Areas** | Binding platform (floating binding panel, scope-badge mechanism), Repeater inspector (`renderSettingsRepeaterOnRepeater()`), CMS data-resolution runtime (`wixData.queryReferenced()` / Blocks `include()`), Repeater context card. |
| **Required Capability Changes** | Add `This item` scope to the binding-picker scope handling. Surface array fields and multi-reference fields of the Parent item's collection at the top of the picker when the selected Repeater is nested. Wire the Nested Repeater's `Items` to a parent-row-scoped query. |
| **Data & Storage Implications** | No new persistence model. The Nested Repeater's binding stores: source collection, field path (Child array or Multi-reference), and an implicit scope marker that the Items source resolves per-Parent-row at render time. |
| **API Implications** | Reuse `wixData.queryReferenced()` for multi-reference resolution and the Blocks `include()` primitive for batched cross-collection fetch. **Do not** introduce a parallel runtime path for canvas binding — wrap the existing primitives. Resolution must be per-Parent-row but batched in a single query per page render where possible. |
| **Permissions & PII Implications** | New design required. Carry forward the parent collection's read-permission model to the Child collection lookup. PII labels on referenced fields must propagate to the rendered Nested Repeater scope. |
| **Observability & Testing Implications** | Log binding-apply events with `outerRepeaterId`, `nestedRepeaterId`, `parentCollection`, `childField`, and resolution-batch size. Add tests for: drop-into-item gesture, picker scope ordering, multi-reference vs. array selection parity, batched fetch correctness. |
| **Dependencies & Rollout Notes** | Depends on the existing `This repeater` scope-badge work being shipped first (or in parallel). Gate behind a feature flag. Roll out to Studio 2 first; mobile-editor parity is a separate spec. |

---

#### FR-002 — Auto-bind Nested Repeater `Items` to first Child array on drop

| Field | Value |
|---|---|
| **ReqID** | FR-002 |
| **Description** | When a Repeater is dropped inside another Repeater's item template and the Parent item exposes one or more array / multi-reference fields, the system auto-binds the Nested Repeater's `Items` to the first available Child array. This mirrors the auto-binding behavior already specified for blank Repeaters (`blank-repeater-prd.md` §5) but scopes the candidate fields to the Parent row context. |
| **Change vs. existing** | Extends the existing auto-bind logic from page-level context attachment to the parent-item-row case. Uses the same field-selection heuristic (first array, with hierarchy labels) and the same "do not overwrite an existing binding" guard. |
| **Scope** | Phase 1 |
| **Priority** | Critical |

##### Use Cases

**UC-1 — Auto-bind on drop into parent item template**

| Element | Value |
|---|---|
| **Use Case Goal** | The builder gets a connected Nested Repeater with zero manual configuration when the Parent item has a single Child array. |
| **Use Case Description** | The builder drops a new (unbound) Repeater into the Outer Repeater's item template. The Outer Repeater is bound and the Parent item has at least one array / multi-reference field. The system silently binds the Nested Repeater's `Items` to the first Child array. The inspector opens with the binding already applied. |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Render parent → child data without manual setup (Linked Intent ID: INT-001 / SUB-001c) |
| **Trigger** | Drop-into-item gesture on a fresh Repeater. |

**Happy Flow:**

1. Builder drags a new Repeater from the Add panel into an Outer Repeater's item template.
2. System detects: Outer is bound, Parent item has ≥1 Child array, dropped Repeater has no `Items` binding.
3. System auto-binds `Items` to the first available Child array.
4. Canvas re-renders with the Nested Repeater populated per Parent row.
5. Inspector opens on the Nested Repeater with the Items row showing the auto-applied binding (with `This item` badge).

**UC-2 — Skip auto-bind when `Items` is already set**

| Element | Value |
|---|---|
| **Use Case Goal** | Auto-binding never silently overwrites an existing binding. |
| **Use Case Description** | The builder pastes or moves a previously-bound Repeater into another Repeater's item template. The system does **not** rebind it; the existing source is preserved (this may produce a context mismatch — see FR-013). |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Predictable behavior with existing work (Linked Intent ID: INT-001 / SUB-001c) |
| **Trigger** | Paste/move of an already-bound Repeater into a parent item template. |

**Happy Flow:**

1. Builder pastes a previously-bound Repeater into the Outer Repeater's item template.
2. System detects: dropped Repeater already has an `Items` binding.
3. System preserves the existing binding.
4. If the existing binding does not match the Parent item's available Child arrays, the inspector flags a context mismatch (handled by FR-013).

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|---|---|---|---|
| Validation & Input | Critical | Parent item has no array / multi-reference fields | Skip auto-bind. Surface the empty-state copy from FR-011. |
| Validation & Input | Important | Parent item has multiple Child arrays of equal "first-ness" | Pick by stable ordering (collection schema order). Builder can change manually. |
| Concurrent & State Changes | Important | Outer Repeater's binding is removed during the drop gesture | Skip auto-bind. Set Nested Repeater to "Outer not connected" (FR-005). |
| Incomplete & Partial States | Nice-to-Have | Builder cancels the drop mid-gesture | No binding is applied. |

**Desired Feelings in Output State:** Delighted (the most common path takes zero clicks); calm (predictable, never surprising — never overwrites).

**Codebase Implications:**

| Aspect | Detail |
|---|---|
| **Impacted Areas** | Drop-handling in the editor canvas, binding platform, `renderSettingsRepeaterOnRepeater()`. |
| **Required Capability Changes** | Detect drop-into-item, evaluate Parent item's Child-array fields, apply binding when conditions are met. |
| **API Implications** | Same as FR-001. |
| **Permissions & PII Implications** | Carry forward FR-001 design. |
| **Observability & Testing Implications** | Track `auto_bind_applied` vs. `auto_bind_skipped` reasons (no_child_array, already_bound, outer_unbound). |
| **Dependencies & Rollout Notes** | Same flag as FR-001. |

---

### 4.2 Inherit Parent Context Automatically (INT-002)

#### FR-003 — Nested Repeater inherits the Parent row context

| Field | Value |
|---|---|
| **ReqID** | FR-003 |
| **Description** | When a Repeater is placed inside another Repeater's item template, it automatically gains access to the Parent row context. This context — the single row of the parent collection that scopes the parent item template — is the binding scope for the `This item` badge in the picker and is the implicit filter on the runtime query. The builder does not perform an attach action to gain access to the Parent row context. |
| **Change vs. existing** | New scope concept added to the binding platform. Sibling to existing page-level contexts and `This repeater` context. Does not change behavior of non-nested Repeaters. |
| **Scope** | Phase 1 |
| **Priority** | Blocker |

##### Use Cases

**UC-1 — Inherit Parent row context on drop or paste**

| Element | Value |
|---|---|
| **Use Case Goal** | The Nested Repeater can immediately bind to fields from the Parent row without manual context attachment. |
| **Use Case Description** | When the builder places a Repeater inside another Repeater's item template, that Repeater inherits the Parent row context as a usable binding scope. No `Attach Context` action is required for the parent's data to be available. |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Inherit parent context (Linked Intent ID: INT-002 / SUB-002a) |
| **Trigger** | Drop, paste, or move of any Repeater into another Repeater's item template. |

**Happy Flow:**

1. Builder drops a Repeater (bound or unbound) into the Outer Repeater's item template.
2. System detects nesting and registers the Parent row context as inherited on the Nested Repeater.
3. Binding picker for any inner element (including the Nested Repeater itself) gains a `This item` scope.
4. Inspector context card on the Nested Repeater shows the inherited Parent row context, visually distinct from directly-attached contexts (see FR-004).

**UC-2 — Re-establish inheritance when the Outer's binding changes**

| Element | Value |
|---|---|
| **Use Case Goal** | When the Outer Repeater's source changes (replace), the Nested Repeater's inherited context updates to reflect the new parent collection. |
| **Use Case Description** | The builder replaces the Outer Repeater's `Items` source. The Nested Repeater's inherited Parent row context now points to the new collection. Inner bindings on the Nested Repeater that referenced the previous parent's fields trigger the existing replace-context warning flow (FR-006 + carry-forward of `blank-repeater-prd.md` §7). |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Predictable inheritance under change (Linked Intent ID: INT-002 / SUB-002a) |
| **Trigger** | Outer Repeater `Items` source replacement. |

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|---|---|---|---|
| Empty States | Critical | Outer Repeater has no `Items` binding | Inherited context is unavailable. Nested Repeater enters "Outer not connected" state (FR-005). `This item` scope is hidden in the picker. |
| Concurrent & State Changes | Important | Builder reorganizes the layout (drags Repeater out of the parent item template) | System removes inherited context. Any inner bindings that depended on it are invalidated; warning surfaces matching the disconnect cascade copy (FR-006). |
| Boundary Conditions | Nice-to-Have | Repeater is inside multiple nested levels (within depth limit) | Inherited context = closest-ancestor Parent row. Deeper ancestors are not exposed. |

**Desired Feelings in Output State:** Empowered, calm.

**Codebase Implications:**

| Aspect | Detail |
|---|---|
| **Impacted Areas** | Binding platform context registry, `renderSettingsRepeaterOnRepeater()`, drop / move / paste handlers. |
| **Required Capability Changes** | Add inherited-context registration on nesting events. Expose context to descendant binding pickers. Invalidate on un-nesting. |
| **Data & Storage Implications** | Inherited contexts are computed, not persisted. No schema change. |
| **API Implications** | None directly; consumed by FR-001's `wixData.queryReferenced()` call. |
| **Permissions & PII Implications** | The inherited context carries the Parent collection's permission and PII tagging. |
| **Observability & Testing Implications** | Test: drop, paste, move, un-nest, deeper nesting up to the depth limit. |
| **Dependencies & Rollout Notes** | Foundation for FR-001, FR-002. Roll out together. |

---

#### FR-004 — Distinguish inherited vs. directly attached contexts in the inspector

| Field | Value |
|---|---|
| **ReqID** | FR-004 |
| **Description** | The Nested Repeater's context card visually distinguishes contexts inherited from the Parent item from contexts directly attached to the Repeater. A direct attach is permitted on top of inheritance, but the inspector makes the relationship and the active binding source clear at a glance. |
| **Change vs. existing** | Extends the existing Repeater context card (today: shows attached contexts on a flat list). Adds an "inherited from parent" indicator and an explicit ordering: inherited first, then directly attached. |
| **Scope** | Phase 1 |
| **Priority** | Critical |

##### Use Cases

**UC-1 — Distinguish inherited vs. directly attached contexts**

| Element | Value |
|---|---|
| **Use Case Goal** | The builder can tell at a glance which contexts are coming from the Parent and which were attached directly. |
| **Use Case Description** | On a Nested Repeater inspector, the context card shows the inherited Parent row context with a clear "from parent item" indication, and any directly-attached contexts in their own group. |
| **Actor** | Studio User / Partner |
| **Intent Binding** | Distinguish inherited vs. attached (Linked Intent ID: INT-002 / SUB-002b) |
| **Trigger** | Selecting the Nested Repeater. |

**UC-2 — Allow direct attach on top of inheritance (mismatch case)**

| Element | Value |
|---|---|
| **Use Case Goal** | The builder can intentionally attach a context that is not the parent's — and see clearly that there is a mismatch. |
| **Use Case Description** | Some valid scenarios require the Nested Repeater's data source to be different from the Parent row context (e.g., a global "Recommended For You" list rendered inside each parent row). The builder can directly attach such a context. The inspector marks the mismatch and shows which source feeds the Nested Repeater's `Items` (handled in detail by FR-013). |
| **Actor** | Studio User / Partner |
| **Intent Binding** | Mismatch case visibility (Linked Intent ID: INT-002 / SUB-002b) |
| **Trigger** | Builder uses Attach Context on a Nested Repeater. |

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|---|---|---|---|
| Concurrent & State Changes | Important | Outer Repeater's source changes; inherited context updates | Inspector context card updates to reflect new inherited source; existing direct attachments stay; mismatch (if any) is recomputed. |
| Boundary Conditions | Nice-to-Have | Many directly-attached contexts (>3) | Card supports collapse / overflow. |

**Desired Feelings in Output State:** Confident, in control.

**Codebase Implications:**

| Aspect | Detail |
|---|---|
| **Impacted Areas** | Repeater context card render path on the Nested Repeater inspector. |
| **Required Capability Changes** | Render two groups (inherited / directly attached) with distinct visual treatment. |
| **API Implications** | None. |
| **Permissions & PII Implications** | Inherited PII labels propagate visually. |
| **Observability & Testing Implications** | Render tests for combinations of inherited + 0/1/many directly-attached contexts. |
| **Dependencies & Rollout Notes** | Depends on FR-003. |

---

#### FR-005 — Show "Outer Repeater not connected" state on Nested Repeater

| Field | Value |
|---|---|
| **ReqID** | FR-005 |
| **Description** | When the Outer Repeater is unbound, the Nested Repeater displays a clear initial state explaining that the Outer must be connected first. The Nested Repeater's `Connect data` action is contextualized — it leads the builder back to the Outer or offers a direct-attach path for advanced cases. |
| **Change vs. existing** | New connection-state on the Nested Repeater inspector. Adapts the existing "not connected" hat copy. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Outer is unbound; Nested Repeater shows guidance**

| Element | Value |
|---|---|
| **Use Case Goal** | Builders are not stuck staring at an empty Nested Repeater wondering why nothing happens. |
| **Use Case Description** | The builder selects the Nested Repeater while the Outer Repeater is unbound. Inspector shows: "Connect the outer Repeater first. Once connected, this inner Repeater can show data from each parent item." Action surface offers a path to the Outer Repeater. |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Clear initial state (Linked Intent ID: INT-002 / SUB-002c) |
| **Trigger** | Selecting a Nested Repeater while its Outer is unbound. |

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|---|---|---|---|
| Empty States | Critical | Outer is unbound and has no available context | System still surfaces the "Outer not connected" guidance; does not silently fail. |
| Permission & Access Control | Important | Builder lacks edit on the Outer Repeater | Action that "selects the Outer" is shown but disabled with explanation. |

**Desired Feelings in Output State:** Confident, oriented.

**Codebase Implications:** Connection-state computation on Nested Repeater (depends on Outer's binding state). No new API.

---

### 4.3 Safely Replace and Disconnect Nested Bindings (INT-003)

#### FR-006 — Warn before Outer-side changes that cascade into nested bindings

| Field | Value |
|---|---|
| **ReqID** | FR-006 |
| **Description** | When a builder replaces or disconnects the Outer Repeater's `Items`, and that change would cascade-disconnect any inner bindings (including the Nested Repeater's `Items` and any grandchild bindings inside the Nested Repeater's items), the system warns the builder before applying the change. The warning includes an accurate count of affected inner bindings — including grandchildren. Cancel preserves all bindings. |
| **Change vs. existing** | Extends [`blank-repeater-prd.md`](../../blank-repeater-prd.md) §7 (replace context warning) and §8 (disconnect Items) to count grandchild bindings. The cascade depth is now 2 — direct inner bindings + bindings inside Nested Repeater items. |
| **Scope** | Phase 1 |
| **Priority** | Critical |

##### Use Cases

**UC-1 — Outer disconnect with grandchild count**

| Element | Value |
|---|---|
| **Use Case Goal** | The builder understands the full cascading effect before disconnecting. |
| **Use Case Description** | The builder triggers disconnect on the Outer Repeater's `Items`. System computes affected bindings: direct inner bindings on the Outer item (existing) + Nested Repeater's `Items` binding + bindings inside the Nested Repeater's item template (new). Confirmation dialog shows total count and lists categories. |
| **Actor** | Studio User / Self-Creator / Partner |
| **Intent Binding** | Safe disconnect with full visibility (Linked Intent ID: INT-003 / SUB-003a) |
| **Trigger** | Builder triggers disconnect on Outer Repeater `Items`. |

**UC-2 — Outer replace with grandchild count**

Same flow as UC-1 but the action is replacing the source.

**UC-3 — Cancel preserves all bindings**

The builder can cancel the dialog at any time; no bindings are changed.

**Edge Cases:**

| Category | Severity | Trigger | Expected System Behavior |
|---|---|---|---|
| Validation & Input | Critical | Inaccurate count surfaced | Treat as a P0 — counts are part of the warning's truth and must be derived from the binding graph, not estimated. |
| Concurrent & State Changes | Important | Another open editor session disconnects the Outer first | The dialog reopens stale; on confirm, system no-ops or reloads the latest state. |
| Boundary Conditions | Nice-to-Have | Zero grandchild bindings to disconnect | Use the existing blank-Repeater PRD §8 copy without the grandchild language. |

**Desired Feelings in Output State:** Confident (no surprise data loss); in control (the user always sees the impact before agreeing).

**Codebase Implications:**

| Aspect | Detail |
|---|---|
| **Impacted Areas** | Existing replace-context and disconnect-Items dialogs; binding-impact computation. |
| **Required Capability Changes** | Walk the binding graph 2 levels deep when computing impact for an Outer-side change. |
| **API Implications** | None new; extends existing impact-counter. |
| **Permissions & PII Implications** | None. |
| **Observability & Testing Implications** | Test scenarios: 0 / 1 / many inner bindings; 0 / 1 / many grandchild bindings; mixed scenarios. |
| **Dependencies & Rollout Notes** | Must ship with FR-001 — there is no shippable v1 without an accurate cascade warning. |

---

#### FR-007 — Disconnect or replace the Nested Repeater independently

| Field | Value |
|---|---|
| **ReqID** | FR-007 |
| **Description** | The builder can disconnect or replace the Nested Repeater's `Items` independently of the Outer Repeater. Confirmation copy describes the cascade scoped to the Nested Repeater (its child-item bindings only) — not the Outer. Layout and styling of both Repeaters are preserved. |
| **Change vs. existing** | Mirrors `blank-repeater-prd.md` §7 (replace) and §8 (disconnect) with the Nested Repeater as the subject. |
| **Scope** | Phase 1 |
| **Priority** | Critical |

##### Use Cases

**UC-1 — Disconnect Nested Repeater `Items`**
**UC-2 — Replace Nested Repeater `Items` source**
**UC-3 — Layout and styling preserved on disconnect / replace**

(Each follows the structure in the blank-Repeater PRD; happy flows preserve outer Repeater data and layout.)

**Edge Cases:** Mirror FR-006 edge cases at one level shallower.

**Codebase Implications:** Reuse existing per-Repeater disconnect / replace path; scope the warning copy to the Nested Repeater.

---

#### FR-008 — Undo immediately after destructive nested-binding changes

| Field | Value |
|---|---|
| **ReqID** | FR-008 |
| **Description** | After any destructive change to a nested binding (Outer disconnect, Outer replace, Nested disconnect, Nested replace), the builder has an immediate undo path that restores the prior binding graph in full — including grandchild bindings that were cascaded. |
| **Change vs. existing** | Extends the editor's existing undo coverage to the binding graph for nested changes. The blank-Repeater PRD calls this out as an open question (`*Open question: do we need revert? maybe we should show a toast with revert action — what is the Editor pattern?*`). This FR closes that question for the nested case. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Undo immediately after disconnect / replace**

The builder can press undo immediately and recover the previous binding graph.

**UC-2 — Persistent undo across the operation**

Undo persists in the editor undo stack (not only as a transient toast) so the builder can recover after additional actions.

**Edge Cases:** Concurrent multi-tab edits — if undo would conflict with a remote change, the system surfaces a conflict resolution path rather than silently overwriting.

**Codebase Implications:** Editor undo coverage for binding-graph operations. No new API.

---

### 4.4 Stay Oriented at the Limits (INT-004)

#### FR-009 — Enforce a depth limit (v1 = 2)

| Field | Value |
|---|---|
| **ReqID** | FR-009 |
| **Description** | v1 supports nesting up to 2 levels (Outer Repeater → Nested Repeater). Attempts to nest a third level via drop or paste are blocked with clear messaging. |
| **Change vs. existing** | New constraint. v1 ceiling matches the most realistic use cases (course → lesson, category → product, meal → dish, season → episode). Webflow ships 3 levels; reserve depth ≥3 for a future phase if data Q4 in [`data-analysis-plan-repeater-in-repeater.md`](../discovery/data-analysis-plan-repeater-in-repeater.md) shows meaningful demand. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Drop blocked at depth 3**

System detects the third-level drop and blocks it with an explanation that v1 supports 2 levels.

**UC-2 — Inspector shows depth at the limit**

When a Nested Repeater is selected at depth 2, inspector states "depth limit reached — adding a Repeater inside this one is not supported in v1."

**Edge Cases:** Boundary — pasting a depth-3 subtree should also be blocked at the topmost violating level.

**Codebase Implications:** Depth check on drop and paste; depth-aware inspector copy.

---

#### FR-010 — Surface items-per-parent cap (v1 = 100)

| Field | Value |
|---|---|
| **ReqID** | FR-010 |
| **Description** | The Nested Repeater renders up to 100 child items per Parent row (Webflow parity). When the resolved Child collection for a Parent row exceeds the cap, the system renders the first 100 (per the runtime sort) and surfaces a clear inspector message that the cap was reached for that parent row. Pagination semantics for Nested Repeaters are deferred to the existing pagination spec ([`product-spec-repeater-pagination.md`](product-spec-repeater-pagination.md)) which already branches via `renderSettingsRepeaterOnRepeater()`. |
| **Change vs. existing** | New cap. Aligns with the runtime's batched-fetch primitive. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Cap surfaced when exceeded**
**UC-2 — Cap not surfaced when not reached**

**Edge Cases:** Boundary at exactly 100 — show no warning. Empty Child array — handled by FR-012.

**Codebase Implications:** Runtime enforcement of the cap; inspector messaging tied to per-parent-row cap status.

---

#### FR-011 — Communicate "no Child arrays available" on Parent item

| Field | Value |
|---|---|
| **ReqID** | FR-011 |
| **Description** | When the Nested Repeater's binding picker is opened and the Parent item's collection has no array or multi-reference fields, the picker shows a clear empty state explaining the situation and offering recovery paths (attach a different context, or modify the parent collection schema). |
| **Change vs. existing** | Extends [`blank-repeater-prd.md`](../../blank-repeater-prd.md) §11 (no array fields available) to the nested case. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Picker empty state for nested case**
**UC-2 — Recovery: attach a different context**

**Codebase Implications:** Binding-picker empty-state for the `This item` scope.

---

#### FR-012 — Render empty Child arrays cleanly per Parent row

| Field | Value |
|---|---|
| **ReqID** | FR-012 |
| **Description** | When a Parent row's Child array exists but is empty (zero entries), the Nested Repeater area for that Parent row renders an empty state without collapsing the Parent row's layout. Other Parent rows with non-empty Child arrays render normally. |
| **Change vs. existing** | New runtime/canvas behavior. The blank-Repeater PRD §12 ("Empty Data After Connection") was deferred — this FR resolves it for the nested case. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Mixed Parent rows: some with children, some empty**

System renders each Parent row independently; empty cases preserve layout slot.

**Edge Cases:** All Parent rows have empty Child arrays — Nested Repeater is uniformly empty per row, but the page is not blank.

**Codebase Implications:** Runtime / canvas rendering for empty inner data; preserve layout reservation.

---

#### FR-013 — Communicate context mismatch on the Nested Repeater inspector

| Field | Value |
|---|---|
| **ReqID** | FR-013 |
| **Description** | When the Nested Repeater's `Items` source is not the inherited Parent row context (mismatch — see FR-004 UC-2), the inspector clearly indicates which source actually feeds the Nested Repeater. The canvas hat reflects the actual `Items` source, not only the inherited context. |
| **Change vs. existing** | Extends [`blank-repeater-prd.md`](../../blank-repeater-prd.md) §10 (Context Mismatch) explicitly to the nested case. |
| **Scope** | Phase 1 |
| **Priority** | Must have |

##### Use Cases

**UC-1 — Mismatch surfaced in inspector**
**UC-2 — Canvas hat reflects actual source**

**Edge Cases:** Validation — when the mismatch makes inner bindings invalid, surface the inner binding's validation error per row.

**Codebase Implications:** Inspector mismatch indicator; canvas-hat resolution.

---

## 5. Non-Functional Requirements

### 5.1 Reliability

#### NFR-001 — Resolution success rate

| Field | Value |
|---|---|
| **ID** | NFR-001 |
| **Category** | Reliability |
| **Description** | The runtime resolution of a Nested Repeater's child rows for any single Parent row succeeds at least 99.9% of the time under normal load. Failures degrade gracefully per FR-001 (placeholder render, retriable indicator) and never collapse the Parent row's layout. |
| **Suggested Threshold** | ≥ 99.9% per-Parent-row resolution success; failure mode constrained to the affected Parent row, never the whole Outer. |
| **Scope / Phase** | Phase 1 — Project-wide for nested-Repeater rendering |
| **Rationale** | Nesting amplifies the blast radius of a transient data error: one bad reference resolution can break the entire Outer if not contained. Containing the failure to one Parent row preserves the user's "confident" feeling at runtime. |

#### NFR-002 — Gradual rollout & rollback

| Field | Value |
|---|---|
| **ID** | NFR-002 |
| **Category** | Reliability |
| **Description** | The nested-Repeater binding flow is feature-flagged at the editor and runtime layer with percentage-based rollout and instant rollback capability. |
| **Suggested Threshold** | 0% → 5% → 25% → 50% → 100% rollout cadence with hold gates; rollback within 5 minutes of a SLO breach. |
| **Scope / Phase** | Phase 1 — Project-wide |
| **Rationale** | Replacing the multi-dataset workaround pattern at scale touches a popular surface; a controllable rollout protects existing sites that rely on the workaround. |

### 5.2 Performance

#### NFR-003 — Page render time with nested rendering

| Field | Value |
|---|---|
| **ID** | NFR-003 |
| **Category** | Performance |
| **Description** | A Live Site page rendering an Outer Repeater with a Nested Repeater of typical depth (2 levels, ≤25 Parent rows × ≤10 child rows each = up to 250 inner items) hits LCP ≤ 2.5s and INP ≤ 200ms on a median Studio 2 visitor device. CHR ≥ 90% on warm caches. |
| **Suggested Threshold** | LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 on Live Sites; Editor canvas first-paint of a nested layout ≤ 1.5s. |
| **Scope / Phase** | Phase 1 — Project-wide; Live Sites + Editor |
| **Rationale** | Bubble's documented "every nested search multiplies DB calls" is the cautionary signal. Wix must batch the cross-collection fetch via the existing `include()` / `queryReferenced()` primitive. Performance is the deciding factor for whether nested Repeaters are usable for catalogs and listings. |

#### NFR-004 — Editor responsiveness during binding apply

| Field | Value |
|---|---|
| **ID** | NFR-004 |
| **Category** | Performance |
| **Description** | The time between the builder pressing `Apply` in the binding panel for a Nested Repeater and the canvas reflecting the new binding (with rows rendered) is ≤ 1.5s p95. |
| **Suggested Threshold** | p95 ≤ 1.5s; p99 ≤ 3s. |
| **Scope / Phase** | Phase 1 — Editor |
| **Rationale** | The "magical first 60 seconds" of binding is what differentiates the new flow from the multi-dataset hack. Slow apply kills the magic. |

### 5.3 Security

#### NFR-005 — Cross-collection authorization

| Field | Value |
|---|---|
| **ID** | NFR-005 |
| **Category** | Security |
| **Description** | The Nested Repeater's per-Parent-row child resolution honors the read-permissions of the Child collection and does not bypass them. PII labels on referenced fields propagate to the rendered Nested Repeater scope. |
| **Suggested Threshold** | 100% of resolution calls authorized against the Child collection's read policy; 0 PII-label leaks. |
| **Scope / Phase** | Phase 1 — Project-wide |
| **Rationale** | Multi-reference rendering crosses collection boundaries — a permission drift here would expose a downstream user's data. |

### 5.4 SEO

#### NFR-006 — Server-side rendering for nested content

| Field | Value |
|---|---|
| **ID** | NFR-006 |
| **Category** | SEO |
| **Description** | Nested Repeater content is server-rendered at the same layer as the Outer Repeater's content, with semantic HTML and structured data (`schema.org/ItemList` or domain-appropriate type) preserved through the nest. |
| **Suggested Threshold** | 100% of Nested Repeater content present in the initial HTML payload of dynamic pages; structured-data validity passes Google's Rich Results test for parent → child shapes (e.g. ProductGroup → Product, Recipe → ingredients). |
| **Scope / Phase** | Phase 1 — Live Sites |
| **Rationale** | Crawler discoverability is a primary value of CMS-driven sites. A nested layout that renders client-side only would silently regress SEO. |

### 5.5 Accessibility

#### NFR-007 — Screen reader and keyboard parity

| Field | Value |
|---|---|
| **ID** | NFR-007 |
| **Category** | Accessibility |
| **Description** | The Nested Repeater meets WCAG 2.1 AA: `Connect data` action and binding picker are keyboard-operable; the inspector context card distinction (inherited vs. attached, FR-004) is announced to screen readers; inspector messaging at limits (FR-009 / FR-010 / FR-011) is announced via live regions. |
| **Suggested Threshold** | Contrast ratio ≥ 4.5:1; 100% of new interactive elements keyboard-reachable; screen-reader announcement verified for all new states. |
| **Scope / Phase** | Phase 1 — Editor; Live Site rendering preserves existing Repeater A11Y |
| **Rationale** | The Editor surface itself must remain accessible — a new flow that regresses the Editor A11Y baseline blocks Studio 2 from agency users with accessibility requirements. |

### 5.6 Localization

#### NFR-008 — All new UI copy externalized

| Field | Value |
|---|---|
| **ID** | NFR-008 |
| **Category** | Localization |
| **Description** | All new editor, inspector, picker, and warning copy introduced by FR-001 through FR-013 is externalized for translation. Right-to-left layout is preserved for the inspector context-card distinction and the binding picker scope ordering. |
| **Suggested Threshold** | 0 hardcoded strings in new code; RTL pass on all new editor surfaces; localization QA before GA in launch locales. |
| **Scope / Phase** | Phase 1 — Editor |
| **Rationale** | Studio 2 is multilingual; new flows that bypass the localization pipeline fragment the editor. |

### 5.7 Usability

#### NFR-009 — Time-to-first-bound-Nested Repeater

| Field | Value |
|---|---|
| **ID** | NFR-009 |
| **Category** | Usability |
| **Description** | A first-time builder, starting from an Outer Repeater bound to a parent collection with at least one Child array, can place and bind a Nested Repeater in ≤ 60 seconds without external help. The drop-into-item gesture with auto-bind (FR-002) achieves this in ≤ 10 seconds. |
| **Suggested Threshold** | p50 time-to-first-bound-Nested-Repeater ≤ 60s; p50 with auto-bind path ≤ 10s. |
| **Scope / Phase** | Phase 1 — Editor |
| **Rationale** | "Render parent → child without leaving the canvas" is the core intent. Anything slower than this means the user reaches for the multi-dataset hack and the feature never replaces it. |

### 5.8 Error Messages

#### NFR-010 — Clear, actionable copy at every limit and failure

| Field | Value |
|---|---|
| **ID** | NFR-010 |
| **Category** | Error Messages |
| **Description** | Every limit-state (depth, items-per-parent, no-array-fields, mismatch, Outer-not-connected, resolution-failed) ships with copy that is plain-language, context-specific, and provides at least one recovery action. No raw error codes are surfaced to the builder. Confirmation copy on disconnect / replace includes accurate counts of affected bindings. |
| **Suggested Threshold** | 100% of new error/limit states reviewed against the Wix UX Writing Glossary; 0 raw error codes; every state has at least one actionable next step. |
| **Scope / Phase** | Phase 1 — Project-wide |
| **Rationale** | This feature replaces three workaround patterns; if the new flow surfaces opaque errors, builders fall back to the workarounds and the platform inherits the worst of both. |

### 5.9 Legal Compliance

#### NFR-011 — PII propagation through nested rendering

| Field | Value |
|---|---|
| **ID** | NFR-011 |
| **Category** | Legal Compliance |
| **Description** | When a Nested Repeater renders fields tagged as PII on the Child collection, the rendered scope inherits the same PII tagging, GDPR / CCPA consent gating, and data-subject-deletion semantics as the Child collection's standalone rendering. |
| **Suggested Threshold** | 100% PII-tag propagation through the nest; legal review of the cross-collection rendering path before GA. |
| **Scope / Phase** | Phase 1 — Project-wide |
| **Rationale** | Nesting introduces a new path through which PII is rendered. A regulatory drift here is a P0 legal exposure. |

### 5.10 Documentation & Education

#### NFR-012 — Help Center coverage at GA

| Field | Value |
|---|---|
| **ID** | NFR-012 |
| **Category** | Documentation & Education |
| **Description** | At GA, the Wix Help Center has at least one canonical user-facing article covering nested Repeaters end-to-end (concept, drop-into-item gesture, picker `This item` scope, depth limit, items-per-parent cap, replace / disconnect cascade). The existing *"CMS Request: Attaching a Repeater onto Another Repeater"* page is updated or replaced. Customer Care is trained before GA on the new flow and how to migrate users from the three workarounds. |
| **Suggested Threshold** | 1 canonical KB article + Customer Care runbook + at least 3 vertical-specific examples (Stores, Restaurants, Bookings) ready at GA. |
| **Scope / Phase** | Phase 1 |
| **Rationale** | The Help Center has carried the request for years; users will look there first when the feature ships. |

### 5.11 Supportability

#### NFR-013 — Observability of nested-binding lifecycle events

| Field | Value |
|---|---|
| **ID** | NFR-013 |
| **Category** | Supportability |
| **Description** | The system emits structured logs for: drop-into-item nesting events, auto-bind apply / skip (with reason), manual-bind apply, inherit / un-inherit, replace / disconnect (with cascade counts), depth-limit and items-per-parent-cap hits. All events carry a correlation ID and are queryable for the data plan in [`data-analysis-plan-repeater-in-repeater.md`](../discovery/data-analysis-plan-repeater-in-repeater.md). |
| **Suggested Threshold** | 100% of FR-001…FR-013 lifecycle events instrumented; logs queryable within ≤ 1 hour of emission. |
| **Scope / Phase** | Phase 1 — Project-wide |
| **Rationale** | The data plan's Q1 (workaround attempts) and Q4 (depth distribution) cannot be answered without these events. Observability is not optional; it is the validation surface. |

---

## 6. Out of Scope

| Item | Reason for deferral |
|---|---|
| AI-assisted nested binding | Scope follows the blank-Repeater PRD's AI deferral. A separate AI section/PRD will cover action-bar AI, AI binding, and AI field-insertion for nested Repeaters. |
| Depth ≥ 3 levels | v1 ships 2 levels. Q4 in the data analysis plan should size demand for depth ≥ 3 before expanding. Webflow's 3-level cap is the natural next stretch. |
| Mobile editor parity | Mobile Repeater has its own API surface. A separate spec covers mobile parity. |
| Tables, Galleries, Charts, and other repeating-but-not-Repeater elements | Out of scope for this PRD. Flagged for follow-up work. |
| Nested Repeaters inside global sections | A **global section** is a reusable Studio 2 block shared across pages. Because a global section has no stable parent data context — it can be placed on any page, inside any Outer Repeater or none — propagating the parent row context into it reliably is out of scope for v1. The `This item` scope will not appear in the binding picker when the Repeater lives inside a global section; the inspector should surface a clear explanation. Matches Webflow's rule: nested Collection Lists cannot live inside Components. Revisit in a later phase. |
| Auto-migration of existing multi-dataset workaround sites | A "convert" affordance is potentially valuable but requires the data plan's Q1 results to size the affected base. Defer until that data is available. |
| Pagination behavior changes for Nested Repeaters | Already specified in [`product-spec-repeater-pagination.md`](product-spec-repeater-pagination.md) via `renderSettingsRepeaterOnRepeater()`. This PRD does not modify pagination semantics. |

---

## 7. Open Questions

| # | Question | Resolution path |
|---|---|---|
| 1 | Items-per-parent cap value — 100 (Webflow parity) vs. tighter / looser | Engineering input on runtime guarantees + data Q4 (depth distribution) |
| 2 | Should the system block the `This item` binding scope inside global sections at the editor layer, or merely degrade at runtime? | Engineering decision; recommend blocking at the editor layer — surface an explanatory message in the inspector rather than a silent runtime failure |
| 3 | Existing-workaround migration — auto-convert, "convert" affordance, or leave alone? | Defer until data plan Q1 sizes the affected base |
| 4 | `onItemUpdated` interplay with nested Repeaters — does the Nested Repeater fire `onItemReady` per Parent row? | Coordinate with the repeater-event-handlers PRD |
| 5 | ~~Wix UX Writing Glossary cross-check on `Nested Repeater`, `Outer Repeater`, `Parent item`, `Child item`, `This item`~~ | **Resolved** — `ux-writing__check_glossary` run on all spec terms. Findings: (1) `Repeater` canonical in Velo domain; use lowercase in UX copy. (2) `Collection`, `Connect`, `Disconnect` canonical/allowed. (3) ⚠️ `item` is a **prohibited variant** for "Cell" in the Studio Editor + Harmony Editor domains ("Use 'Cell' instead of 'item' when referring to grid children") — our "Parent item" / "Child item" refer to Repeater rows, not grid cells, so usage is contextually different; requires UX Writing team to formally confirm the carve-out. (4) `Nested Repeater`, `Outer Repeater`, `Parent item`, `Child item`, `This item`, `Child array`, `Child collection`, `Connect data` — **no existing glossary entries** — all 8 terms must be formally submitted to the UX Writing team for addition before GA. |
| 6 | ~~Whether `[NEW]` / `[MODIFY]` / `[EXISTS]` delta tagging should be added once `ck-product-current-impl` runs~~ | **Resolved** — delta tags added to §3 Summarization Table. Source: codebase scan of `wix-private/odeditor-packages` HEAD `8b0a2f9` (2026-04-26). Note: `CMS/propertyBindingPanel` source lives in external CMS packages (`@wix/wix-data-client-editor3-packages` etc.), not in `odeditor-packages` — delta tags for FR-001, FR-011 reflect this. |
| 7 | Jira UPI / EDITOR / CMS / STUDIO scrape for prior tickets — Jira auth was unavailable during discovery | Auth still required — sign in at https://mcp-s-connect.wewix.net/auth/jira, then re-run a search across EDITOR / CMS / STUDIO projects for `text ~ "nested repeater" OR text ~ "repeater on repeater" OR text ~ "repeater in repeater"` |

---

## 8. Glossary

Terminology aligns with [`terminology-research-repeater-in-repeater.md`](../discovery/terminology-research-repeater-in-repeater.md).

| Term | Definition |
|---|---|
| **Repeater** | The visual component that repeats item UI. |
| **Nested Repeater** | A Repeater placed inside another Repeater's item template, bound to a Child array of the Parent item. |
| **Outer Repeater** | The enclosing Repeater whose item template contains the Nested Repeater. |
| **Parent item** | A single rendered row of the Outer Repeater. |
| **Child item** | A rendered row inside a Nested Repeater. |
| **Child array** | A list-shaped field on the parent collection that the Nested Repeater binds to (general term). |
| **Child collection** | A multi-reference field connecting parent and child collections (specific term for the multi-reference data shape). |
| **Parent row context** | The single data row of the parent collection that scopes everything inside the Outer Repeater's item template. |
| **`This item` (scope badge)** | A binding-picker scope label that surfaces fields from the Parent row context. Sibling to `This repeater` and page-level scopes. |
| **`Items` property** | The Repeater's data property. Binds to an array (or, in the nested case, a Child array of a Parent item). |
| **Inherited context** | A context that the Nested Repeater gains automatically by being placed inside the Outer Repeater's item template. No explicit attach action. |
| **Directly-attached context** | A context the builder explicitly attaches to the Nested Repeater (sibling of inheritance; permitted on top of inheritance — see FR-004). |

---

**Saved to:** `artifacts/product/product-spec-repeater-in-repeater.md`

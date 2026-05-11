# Internal Discovery Report: Repeater Event Handlers

## How It Works Today

In the current Wix Studio editor, a Repeater is a layout element that displays a list of items using a shared template design. Users can add a Repeater from the Layout Tools, then manage items (duplicate, rename, reorder, delete) from the Inspector panel. Items can be populated in three ways: statically (each item edited directly on the canvas), via a connected dataset (CMS collection drives content), or via Velo code (`.data` property is assigned an array).

From a developer's perspective, the Velo API exposes the following on the Repeater:

- **`onItemReady($item, itemData, index)`** — fires when new items are created; the `$item` selector is scoped to the item being rendered. Can be added as a static event handler via the Properties & Events panel (Code panel, not Inspector), or as a dynamic handler with `$w('#repeater').onItemReady(...)`.
- **`onItemRemoved($item, itemData, index)`** — fires when items are removed by data update; does NOT fire for static editor-defined items.
- **`forEachItem($item, itemData, index)`** — iterate over all current items programmatically; called manually after data updates since `onItemReady` doesn't re-fire for existing items.
- **`forItems([ids], $item, itemData, index)`** — iterate specific items by ID array.
- **`data`** property — get/set the Repeater's data array; setting it creates/removes items but does not re-trigger `onItemReady` for existing item IDs.

The Studio Editor Inspector panel for Repeater shows design, layout, and management controls only. **Event handlers are not surfaced in the Inspector.** They are accessible only in the Code panel (Properties & Events panel in the Velo workspace). This is an open, unresolved feature request — "Element Properties and Events in the Inspector Panel" — currently listed as "Collecting votes" with no committed timeline ([support.wix.com](https://support.wix.com/en/article/studio-editor-request-element-properties-and-events-in-the-inspector-panel)).

For child elements inside the Repeater template (buttons, images, text), standard interaction events (click, hover, focus) can be wired in the Properties & Events panel and will fire with item context accessible via `event.context.itemId` and `$w.at(event.context)`.

## Problem

The Repeater's current event model has four concrete gaps for Studio 2:

1. **Missing `onItemUpdated`** — setting `.data` on an existing Repeater item fires no event. Developers must manually call `forEachItem()` or `forItems()` as boilerplate after every data update.
2. **Broken item scoping** — the Sept 2024 legacy→dynamic event handler migration broke the `$item` selector inside event handlers, causing it to always return the first item's data. The workaround (`$w.at(event.context)`) works but is non-obvious and not documented prominently.
3. **Hover/mouse events silently fail on dataset-connected Repeaters** — `onMouseIn`/`onMouseOut` on child elements stop working when the Repeater is connected to a CMS dataset.
4. **No Repeater events in the Inspector panel** — `onItemReady` and `onItemRemoved` are code-panel-only. Studio 2's goal of making event handlers accessible without writing code is blocked by this gap.

## Current State

### Key Insights

- **`onItemReady` IS already supported as a static panel handler** (confirmed by dev.wix.com docs) — but only in the Code panel, not the Inspector. The infrastructure to wire it visually exists; the surface needs to move.
- **The `$item` regression from Sept 2024 is unconfirmed as fixed.** If still live, any Studio 2 spec that relies on item-scoped code generation will build on a broken foundation. This is a blocking dependency.
- **`onItemUpdated` is a genuine API gap, not just a panel gap.** Adding it requires platform work, not just a UI change.
- **Wix Blocks has a "Dynamic Repeater"** variant with a slightly different architecture (code-only, no canvas editing per item) — this may have a cleaner event contract that Studio 2 can reference or align with.
- **The Inspector panel feature request is unresolved.** Studio 2 has the opportunity to ship this natively as part of defining the Repeater component — it would close an open community ask.

---

## Platform Landscape

### Relevant Verticals

| Vertical | What It Does | Relevance to Problem |
|----------|-------------|---------------------|
| Velo / $w API | JavaScript framework for Wix sites; all element events go through it | Repeater's entire event model is defined here; any change requires Velo/platform team involvement |
| CMS / Datasets | Connects repeating elements to data collections | Source of the hover event bug; dataset connection changes Repeater behavior in undocumented ways |
| Wix Blocks | App/widget building framework with Dynamic Repeater variant | Alternative Repeater architecture that may have a cleaner event model for Studio 2 to reference |
| Studio Editor (current) | The existing Wix Studio editor | The context being replaced/redesigned; its Inspector panel gap is what Studio 2 must solve |

### Existing Capabilities

| Capability | What It Does | Relevance to Problem |
|-----------|-------------|---------------------|
| `onItemReady()` (static handler) | Fires when new Repeater items are created; already wirable from Properties & Events panel | Must-expose in Studio 2 Inspector; infrastructure exists, surface needs moving |
| `onItemRemoved()` | Fires when Repeater items are removed via data update | Must-expose in Studio 2 Inspector; has the static-item exception that should be documented |
| `forEachItem()` / `forItems()` | Manually iterate all/specific items to apply updates | Developer workaround for missing `onItemUpdated`; may be surfaceable as a panel action in Studio 2 |
| `$w.at(event.context)` | Returns a correctly scoped `$item` selector for the interacted item | Current workaround for broken `$item`; Studio 2 should generate this pattern by default in code stubs |
| Standard child-element events (click, hover, etc.) | Per-element events inside Repeater template wirable via Properties & Events panel | Already available; item context passes via `event.context.itemId`; Studio 2 should make this the default entry point |
| Properties & Events panel (Code panel) | Visual panel for wiring event handlers without writing code | Currently inaccessible from Inspector; Studio 2 should integrate this into the component inspector |
| AB Pattern interactions (Hover/Click only) | Hover and click interactions available for AB-patterned Repeater items in Studio | Limited interaction model; Studio 2 needs to expand beyond AB pattern constraints |

### Adjacent Platform Services

| Service | What It Does | Relevance to Problem |
|---------|-------------|---------------------|
| Wix Blocks Dynamic Repeater | App-building variant of Repeater; code-only, CMS-connected, no per-item canvas editing | Different architecture; may have resolved some event scoping issues; worth internal comparison before speccing Studio 2 |
| Mobile Repeater (`$w/mobile-repeater`) | Separate mobile API for Repeater | Has its own API surface; Studio 2 spec should clarify whether mobile Repeater shares the same event model |
| Dataset / WixData API | Drives CMS-connected Repeaters | The source of data changes; an `onItemUpdated` event would need to be wired here — when the dataset pushes a data change to the Repeater, that change needs to trigger the event |

### Potential Extensions

| Area | What It Does Today | How It Could Be Extended |
|------|-------------------|------------------------|
| Properties & Events panel | Exposes event handlers in Code panel (not Inspector) | Move/replicate Repeater-specific events into Studio 2 Inspector; already has the wiring infrastructure |
| `onItemReady` → `onItemUpdated` | Lifecycle only covers creation | Extend with `onItemUpdated` callback triggered when an existing item's data changes — requires platform API work |
| Child-element event stubs | Code stubs use `$w` selector by default | Generate `$item`-scoped stubs by default when wiring events inside a Repeater — design decision, low effort if code generation layer supports it |

---

## Ranked Solutions

### 1. Surface `onItemReady` and `onItemRemoved` in Studio 2 Inspector
- **Location:** [dev.wix.com — onItemReady](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater/on-item-ready) | [onItemRemoved](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater/on-item-removed)
- **Why This Might Help:** Both handlers already exist in the Velo API and are already wirable from the Properties & Events panel in the Code workspace. Studio 2 needs to expose them in the Inspector when a Repeater is selected — this is an UI surface change, not an API change. It directly delivers the stated goal (define which event handlers to expose for Repeater in Studio 2).
- **Maturity:** Production
- **Effort to Adopt:** Low — the API and panel infrastructure exist; this is a surface/UI decision
- **Limitations:** Does not address `onItemUpdated` (missing from API) or the `$item` scoping regression.

### 2. Default to `$item`-scoped code stubs for Repeater child-element events
- **Location:** [dev.wix.com — Working with Repeaters](https://dev.wix.com/docs/velo/articles/getting-started/working-with-repeaters)
- **Why This Might Help:** The most common developer mistake with Repeater events is using `$w` instead of `$item` inside `onItemReady`. If Studio 2's code generation defaults to `$item` (or `$w.at(event.context)`) whenever a developer wires a child-element event inside a Repeater, the entire scoping problem becomes opt-out rather than opt-in. This is achievable as a code generation policy.
- **Maturity:** Internal design decision — no new API required
- **Effort to Adopt:** Medium — requires Studio 2's code generation/stub layer to be aware of Repeater context
- **Limitations:** Only helps developers who use the visual wiring panel; doesn't help those writing code directly.

### 3. Expose `forItems()` as a panel-level "Update item" action
- **Location:** [dev.wix.com — forItems](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater/for-items)
- **Why This Might Help:** `forItems()` is the current workaround for the missing `onItemUpdated` — developers call it manually after updating `.data`. Surfacing it as a studio-level action (e.g., triggered when a bound data field changes) could partially close the `onItemUpdated` gap without requiring a new platform API. It bridges the gap while the full `onItemUpdated` lifecycle event is developed.
- **Maturity:** Production (API); not yet surfaced in Studio panels
- **Effort to Adopt:** Medium — requires Studio 2 panel design work
- **Limitations:** A workaround, not a real solution; `onItemUpdated` as a true lifecycle event is still the right long-term answer.

### 4. Align with Wix Blocks Dynamic Repeater event model
- **Location:** [dev.wix.com — Dynamic Repeaters in Blocks](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/dynamic-repeaters-in-blocks)
- **Why This Might Help:** Wix Blocks has its own Dynamic Repeater architecture, built for apps rather than site-building. It may have resolved some event scoping issues or have a cleaner contract. Before defining the Studio 2 Repeater event model from scratch, comparing with the Blocks Dynamic Repeater model could reveal reusable patterns or avoid architectural divergence.
- **Maturity:** Production (Blocks framework)
- **Effort to Adopt:** Low for research; Medium for alignment if architectures diverge
- **Limitations:** Blocks Dynamic Repeater is app-building-focused; Studio 2 Repeater is site-building-focused; they may have intentionally different models.

---

## Technical Risk

| Risk | Already Addressed? | New Work Required |
|------|-------------------|-------------------|
| `$item` selector regression from Sept 2024 migration | Unknown — status unconfirmed | Verify with Velo/platform team; if not fixed, Studio 2 spec cannot rely on `$item` pattern working reliably |
| `onItemUpdated` requires new platform API | No — the event doesn't exist in Velo today | Platform/Velo team work to introduce the callback; not a UI-only change |
| Hover/mouse events breaking with dataset-connected Repeater | Unknown — may be a dataset-layer bug | Root cause investigation needed (dataset layer vs. Repeater event system) — Studio 2 PM should confirm with platform team |
| Inspector panel not surfacing events (open feature request) | No — still "Collecting votes" | Studio 2 could own this as part of component definition; would need editor infrastructure work |

---

## Open Questions

- Is the Sept 2024 `$item` scoping regression fixed, still live, or intentional behavior? This is blocking for the Studio 2 event contract.
- Does the Wix Blocks Dynamic Repeater have a different/better event model that Studio 2 should align with or diverge from deliberately?
- Is `onItemUpdated` on the Velo roadmap, or is it purely in scope for the CMS/Studio 2 group to define and request?
- Which team owns the Properties & Events panel in Studio 2 — the CMS group or the editor infrastructure group? This determines who can implement the Inspector surface changes.
- Does the Mobile Repeater (`$w/mobile-repeater`) share the same event model, or does Studio 2 need to spec it separately?

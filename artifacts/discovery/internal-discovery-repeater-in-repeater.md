# Internal Discovery — Repeater in Repeater (Studio 2 CMS)

**Project:** Repeater in Repeater (Studio 2 CMS)
**Phase:** Discovery — Wix Internal
**Date:** 2026-05-06

---

## How It Works Today

In the current Wix Studio editor, a Repeater binds to a single CMS collection (or array data source) at a time via the `Items` property. The binding is **flat** — when the user drops a second Repeater inside a parent Repeater's item template, the inner Repeater does **not** inherit the parent's row context. Its `Items` picker still offers page-level contexts, not "this item's array fields."

Builders therefore reach for one of three workarounds:

1. **Multi-dataset filter chaining.** Add a secondary dataset on the page, configure it to filter by the parent's reference field, connect the inner Repeater to the secondary dataset. Documented in the Help Center but brittle, and requires the user to think in datasets.
2. **Side-by-side filtered Repeaters.** Place multiple Repeaters on the same page bound to the same dataset with different filters. **Fails** — the dataset is shared and filters cascade. Drives the 76-mention "shared dataset" support cluster and the 119-mention "multiple repeaters instead of a unified structure" spacing cluster.
3. **Velo / Wix Blocks code.** Use `$w('#outer').onItemReady` + `wixData.queryReferenced()` + `$w('#inner').data = …`, or — in Wix Blocks — wrap the inner Repeater in a widget and use `include()` + prop-drilling.

The current prototype (`pages/context-binding-prototype/binding-platform-demo.html`) already includes a **separate inspector render path for nested Repeaters** — `renderSettingsRepeaterOnRepeater()`. This is acknowledged in [`product-spec-repeater-pagination.md`](../product/product-spec-repeater-pagination.md): pagination has two render paths, "`renderSettingsRepeaterOnRepeater()` (nested repeaters)" and "`renderSettings()` (standard repeaters)." So the editor team has already started branching behavior for the nested case — but only for pagination, not for the binding model itself.

The existing [`blank-repeater-prd.md`](../../blank-repeater-prd.md) covers connection for the top-level Repeater. It mentions nested arrays as selectable types ("Nested arrays and multi-reference fields can appear as selectable array options. Array labels should show enough hierarchy to distinguish nested fields"), but does **not** specify the binding flow when the Repeater being connected is itself inside another Repeater item.

## Problem

Three concrete gaps for Studio 2:

1. **Binding model is flat.** The inner Repeater's `Items` picker has no concept of "the parent item's array/multi-reference fields." There is no `This item` scope badge analogous to the existing `This repeater` scope.
2. **Inspector branches for pagination but not binding.** `renderSettingsRepeaterOnRepeater()` exists for pagination tweaks, but binding/disconnect/replace flows reuse the standard render path that wasn't designed for the nested case.
3. **No canvas affordance for the parent→child relationship.** A user dragging a Repeater into another Repeater's item gets no signal that the inner Repeater can be wired to a child collection. Webflow's drag-into-item paradigm is the gold standard; Studio 2 currently has zero of that signal.

## Current State

### Key Insights

- **The runtime can do it.** Wix Blocks already supports nested dynamic Repeaters via `wixData.queryReferenced()` / `include()` and widget prop-drilling ([Nest Dynamic Repeaters dev docs](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/nest-dynamic-repeaters)). Studio 2 should reuse this primitive, not build a parallel one.
- **The pagination spec already branches for nested.** `renderSettingsRepeaterOnRepeater()` is precedent — the editor team has accepted that nested Repeaters need their own inspector treatment in some areas. Extend that pattern to binding.
- **Multi-reference is the data shape that maps cleanly.** The 108-mention "multi-reference field not displaying dynamic data in repeater" cluster is the data plumbing under nested. Fixing nested without fixing this leaves users stuck.
- **The Help Center has a literal "request" page.** ["CMS Request: Attaching a Repeater onto Another Repeater"](https://support.wix.com/en/article/cms-request-attaching-a-repeater-onto-another-repeater) is Wix-acknowledged framing — *"This feature would allow you to group items within a repeater based on a reference field. For example, you would be able to group dishes within the main repeater by meal."*
- **The blank-Repeater PRD is the model.** Match its depth (initial state, connect, replace, disconnect, edge cases). Reuse its `Apply` flow and confirmation copy patterns where possible.

---

## Platform Landscape

### Relevant Verticals

| Vertical | What It Does | Relevance to Problem |
|----------|-------------|---------------------|
| Studio 2 Editor (the surface) | Visual builder where Repeaters live and bind | Where the new flow has to land — canvas, inspector, action bar |
| Wix Data / CMS | Collections, fields, multi-reference, datasets | Source data shape; multi-reference is the natural binding for nested |
| Wix Blocks | App/widget builder with `include()` + prop-drilling | Has the runtime primitive working; Studio 2 should wrap it |
| Velo / `$w` API | Code path for Repeater data and events | Power-user fallback today; should remain available but should not be the only path |
| Binding Platform (existing prototype) | Floating binding panel, scope badges, context cards | Where `This item` scope must be added |

### Existing Capabilities

| Capability | What It Does | Relevance to Problem |
|---|---|---|
| `Items` binding picker | Lets the user select context + array field | Needs a `This item` scope option when the Repeater is inside another Repeater |
| Floating binding panel | Modal-style picker for selecting context + field | Already supports auto-select-first-array; extend to surface parent-item arrays |
| Repeater context card | Shows attached context per Repeater | Inner Repeater's card should distinguish "inherited from parent" vs. "directly attached" |
| `renderSettingsRepeaterOnRepeater()` | Inspector render path for nested-Repeater pagination | Precedent for branching; extend for binding & disconnect |
| `wixData.queryReferenced()` | Runtime API for fetching referenced rows | Backbone for inner Repeater fetch — used by Blocks, can drive canvas |
| `include()` (Blocks) | Eager-load referenced collection in one query | Avoids N+1 queries; Studio runtime should reuse |
| Multi-reference fields in CMS | Many-to-many relationships between collections | The data shape for nested |
| Action bar / inspector hat | Disconnected-state messaging on Repeater | Needs equivalent state for "no parent context available" |
| Replace context warning | Warns when inner bindings will break on switch | Pattern from blank-Repeater PRD; carries over for nested replace |
| Disconnect confirmation dialog | Confirms cascading destruction of inner bindings | Same — extend for nested-disconnect cascade |

### Adjacent Platform Services

| Service | What It Does | Relevance to Problem |
|---|---|---|
| Dataset / WixData API | Drives CMS-connected Repeaters | Multi-dataset hack lives here; nested should bypass and use the parent-row primitive |
| Mobile Repeater (`$w/mobile-repeater`) | Separate mobile API surface | Confirm whether nested binding is shared or duplicated |
| Wix Blocks runtime | App-level dynamic Repeaters with `include()` | Reference implementation; align Studio canvas runtime with this |
| Wix Stores / Restaurants / Bookings / Blog | Native verticals with parent→child shape (categories→products, menus→dishes, services→staff, categories→posts) | Templates that should ship with nested-Repeater on day one — biggest demo wins |
| Site Generator (SG) | AI-driven site creation flow | If nested Repeaters become first-class, SG should be able to generate them for catalog/listing prompts |

### Potential Extensions

| Area | What It Does Today | How It Could Be Extended |
|---|---|---|
| Binding picker scopes | `This repeater`, page-level, parent-context scopes | Add `This item` as a new scope when the Repeater is nested; auto-rank parent-item array/multi-reference fields to the top |
| Repeater context card | Shows attached contexts | Distinguish "inherited from parent item" with explicit visual treatment |
| Inspector header / hat | Shows connection state | New disconnected state: "outer Repeater not connected — connect parent first" |
| Action bar `Connect data` button | Opens binding panel | When inside a parent item, default the picker to the parent's child arrays |
| Pagination spec | Already branches for nested via `renderSettingsRepeaterOnRepeater()` | Extend the same branching pattern to binding/disconnect/replace |
| Auto-bind | Auto-binds first array when context is attached | When the user drops a Repeater into another Repeater's item, auto-bind to the first available parent-item child array |

---

## Solution Discovery

### Reusable patterns from existing PRDs

- **Initial-state messaging** ([`blank-repeater-prd.md`](../../blank-repeater-prd.md) §1) — extend "not connected" copy for the nested case to: "this Repeater is inside another Repeater — connect the outer one first" when the parent is unbound.
- **Floating binding panel** (`blank-repeater-prd.md` §3) — same panel, with `This item` scope added.
- **Auto-select first array** (`blank-repeater-prd.md` §5) — same logic, scoped to parent-item arrays/multi-reference fields when nested.
- **Replace context warning** (`blank-repeater-prd.md` §7) — same pattern; the count of affected inner bindings now must include grandchild bindings.
- **Disconnect cascade** (`blank-repeater-prd.md` §8) — when the **outer** Repeater's `Items` is disconnected, the inner Repeater's binding cascades. The confirmation copy must call this out explicitly.
- **`This repeater` scope badge** (`blank-repeater-prd.md` §3) — extend to `This item` for nested.
- **Pagination branch** ([`product-spec-repeater-pagination.md`](../product/product-spec-repeater-pagination.md)) — `renderSettingsRepeaterOnRepeater()` precedent.

### Open dependencies

- **Multi-reference data path performance.** Confirm with engineering that `include()` / `queryReferenced()` is the right primitive at canvas runtime and that it batches — Bubble's nested-list performance horror story (every nest multiplies DB calls) must be avoided.
- **Depth limit decision.** Recommend 2 for v1; Webflow's 3-level cap is the next stretch. Engineering input needed on what the runtime can guarantee.
- **Mobile parity.** Whether mobile Repeater shares the same nested binding path or needs its own spec.
- **Component / global block restriction.** Webflow blocks nested Collection Lists inside components — does Studio 2 follow suit? Recommend yes for v1.

### Internal artifacts to reference in the PRD

- [`blank-repeater-prd.md`](../../blank-repeater-prd.md) — connection model, scope badges, replace/disconnect patterns.
- [`product-spec-repeater-pagination.md`](../product/product-spec-repeater-pagination.md) — nested-render-path precedent.
- [`repeater-context-meeting-checklist.md`](../../repeater-context-meeting-checklist.md) — meeting notes covering open questions on context attachment, item-level settings, switch experience.
- [`pages/context-binding-prototype/binding-platform-demo.html`](../../pages/context-binding-prototype/binding-platform-demo.html) — working prototype that already implements `renderSettingsRepeaterOnRepeater()`.
- Existing event-handlers discovery set under [`artifacts/discovery/`](.) — covers `onItemReady`, `$item` scoping, and Inspector-vs-Code-panel split (relevant when nested Repeaters wire events inside child items).

### Out of scope

- AI-assisted nested binding flows — defer to a separate AI section as the blank-Repeater PRD already does.
- Nested binding for Tables, Galleries, Charts, and other repeating-but-not-Repeater elements — flag for follow-up; out of scope for this PRD.
- Cross-collection joins beyond reference / multi-reference — out of scope.

---

## Recommended Next Steps

1. **Spec the binding-picker `This item` scope** as a follow-up to the existing `This repeater` work. Same surface, one additional scope, parent-item array / multi-reference fields surfaced first.
2. **Extend `renderSettingsRepeaterOnRepeater()`** beyond pagination to cover the binding/disconnect/replace flows for nested Repeaters.
3. **Confirm with the Blocks team** that `include()` is the right runtime primitive to wrap in canvas-binding.
4. **Pick depth = 2 for v1**, document it explicitly in the PRD, surface the limit in the inspector when reached.
5. **Ship template wins on day one** — Stores categories→products, Restaurants menus→dishes, Bookings services→staff, Blog categories→posts, generic CMS parent→child.

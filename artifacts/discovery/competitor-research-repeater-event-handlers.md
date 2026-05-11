# Competitive Snapshot: Repeater Event Handlers — Studio 2 Editor

How web editors handle event models for repeating/dynamic-list components, to inform which event handlers Studio 2 should expose when a Repeater is selected.

## Landscape Map

| # | Name | Type | Has Feature? | Approach (1-liner) |
|---|------|------|-------------|-------------------|
| 1 | **Webflow** | Direct | Partial | Visual interaction triggers (click, hover, scroll) on Collection List items, but per-item scoping in CMS lists is non-intuitive and requires manual selector tricks |
| 2 | **Framer** | Direct | Yes | Code components expose event handlers as visual panel controls via `EventHandler` property control; variant-based states handle interaction model per item |
| 3 | **Bubble** | Direct | Partial | Click events work per Repeating Group cell natively; hover requires a Reusable Element workaround — no direct hover trigger on cells |
| 4 | **Elementor (WordPress)** | Adjacent | Yes (API-level) | Developer-facing widget API exposes lifecycle events (add/duplicate/delete item) for repeater controls; not a user-facing interaction model |
| 5 | **React / Next.js** | Alternative | Yes (code-only) | Full native event system with trivial per-item scoping (`map` + closure); no visual panel — the reference implementation developers fall back to |
| 6 | **Custom HTML/JS embed in Wix** | Substitute | Yes | Developers bypass Velo entirely; workaround for missing events — signals unmet need |
| 7 | **Builder.io** | Adjacent | Partial | Component-based visual editor with some event support; per-item model not well-documented publicly |
| 8 | **Wix Studio Classic (Velo)** | Reference | Partial | `onItemReady` + `onItemRemoved` exist; `onItemUpdated` missing; item scoping (`$item`) buggy post-Sept 2024 update |

---

## Summary Comparison (Top 3)

| Dimension | Webflow | Framer | Bubble |
|-----------|---------|--------|--------|
| **How they solve it** | Visual interaction triggers scoped via ancestor/descendant/sibling selectors; applies to Collection List items | Component variants + `EventHandler` property control exposed in panel; code components are first-class | Per-cell workflow triggers for click; hover requires Reusable Element workaround |
| **Key differentiator** | Most mature visual interaction system; GSAP replatform (2025) adds pro-level code power | Event handlers exposed directly in panel for code components; variant state model is clean and scalable | No-code workflow model makes every element action-capable without writing code |
| **Biggest gap** | Affecting siblings of a hovered Collection List item requires non-obvious manual selector tricks; frequently reported as confusing by Webflow users ([forum](https://discourse.webflow.com/t/hover-interactions-to-effect-each-item-in-collection-list-independent-from-the-others/178979)) | CMS Collection List items don't have per-item event panel controls (only code components do); interaction model for dynamic lists is less documented | Hover on Repeating Group cell requires Reusable Element workaround ([Bubble forum](https://forum.bubble.io/t/setting-a-state-when-repeating-groups-cell-is-hovered/151378)); no native hover trigger per cell |
| **Feature gating** | Interactions on all plans; GSAP/advanced code: paid plans | Visual interactions: all plans; CMS: paid plans | Workflows: all plans; advanced features: paid |
| **User sentiment** | Mixed — interactions are powerful but CMS list scoping frustrates developers ([Webflow forum](https://discourse.webflow.com/t/applying-a-hover-effect-to-collection-items/251434)) | Positive for component authors; CMS interaction model praised for data binding; less praise for dynamic-list event specifics | Mixed — hover limitation is well-known and frequently discussed; click per cell is praised |

---

## Feature Comparison

| Dimension | Webflow | Framer | Bubble |
|-----------|---------|--------|--------|
| **Entry point** | Select Collection List item template → Interactions panel → Add trigger | Select code component → Properties panel → `EventHandler` control | Select element inside Repeating Group → Workflow tab → Add event |
| **Item lifecycle events** | None exposed in panel (no `onItemReady` equivalent) | None for CMS lists; code component lifecycle via React props | None for item creation/removal; workflow triggers are interaction-only |
| **Item interaction triggers** | Click, Hover, Mouse move, Scroll into view — all available in panel; scope selector determines which item is affected | Variant states (Default → Hover → Selected → etc.); transitions between states are the event model | Click, Focus — native per cell; Hover — requires Reusable Element workaround |
| **Item context (identity)** | Scope selectors ("First ancestor", "Descendants") handle targeting — not explicit item ID | Variant state machine is item-scoped by default (each component instance has its own state) | `Current cell's` reference pattern — Bubble knows which cell triggered the event natively |
| **Dataset-connected lists** | Interactions work regardless of CMS binding; data and interactions are decoupled | Data binding and interactions are decoupled; no degradation when CMS-connected | Workflows fire regardless of data source |
| **Code stubs generated** | None — interactions are visual-only; no generated code | Auto-generates TypeScript/React when you add interaction to code component | None — no-code model |
| **Notable tricks** | GSAP integration (2025): developers write custom GSAP animations alongside visual interactions — no either/or choice ([Webflow docs](https://help.webflow.com/hc/en-us/articles/42832301823635-Intro-to-Interactions-with-GSAP)) | `EventHandler` property control makes event handlers inspectable and overridable in the panel — a code component author sets the defaults, a non-coder can override the handler from the panel ([Framer docs](https://www.framer.com/developers/property-controls)) | "Current cell's" reference pattern is a clean mental model — every data reference inside a Repeating Group automatically knows its row context |

---

## User Sentiment

| Signal | Webflow | Framer | Bubble |
|--------|---------|--------|--------|
| **Praised** | Visual power + no-code; GSAP integration praised by power users | Component event model praised; variant states feel natural | Click-per-cell works out of the box; "Current cell's" pattern intuitive |
| **Complained about** | Per-item scoping in CMS lists is frequently confusing; "affect siblings" not supported for nested dynamic items ([Webflow forum](https://discourse.webflow.com/t/nested-sibling-interactions-on-dynamic-lists/43986)) | CMS collection list interactions less powerful than code components; per-item event model not well-surfaced for non-coders | Hover on Repeating Group requires workaround; community has posted the same workaround dozens of times ([Bubble forum](https://forum.bubble.io/t/a-hover-on-a-cell-of-a-repeating-group/529)) |
| **Requested** | Native "affect only this item's siblings" for CMS lists | Per-item interaction states for CMS lists without requiring code components | Native hover trigger per Repeating Group cell |
| **Workarounds** | Manual CSS class toggling via JavaScript embed; interaction scope selectors with ancestor targeting | Wrap CMS list items in a code component to get event handler controls | Reusable Element inside Repeating Group cell to get Group Focus hover behavior |

---

## Top 3 Insights

1. **Per-item event scoping is the universal pain point across all editors.** Webflow, Bubble, and Wix all struggle with it in dynamic lists. Only Framer solves it cleanly — by making each component instance its own state machine. Studio 2 has a real opportunity to be the first visual editor where item-scoped events work reliably out of the box, without workarounds.

2. **Lifecycle events are a Wix-specific concept — no competitor exposes them in a visual panel.** Webflow and Framer treat list interactions as user-triggered (click, hover). Only Wix (via Velo) exposes `onItemReady` and `onItemRemoved` as panel events — this is genuinely differentiated. The gap is that `onItemUpdated` is missing, and the existing lifecycle hooks don't appear reliably in the Studio editor panel. Studio 2 should make all three lifecycle events (`onItemReady`, `onItemUpdated`, `onItemRemoved`) first-class panel events.

3. **Framer's `EventHandler` property control is the best model for what Studio 2 wants.** Code component authors define the event contract; the panel exposes it visually; non-coders can override the handler. If Studio 2 generates `$item`-scoped event stubs by default (instead of requiring developers to know the `$w.at(context)` workaround), that mirrors Framer's "right default, overridable" approach — and closes the most common developer mistake in one shot.

## Biggest Opportunity

> **Studio 2 can be the first visual editor with a reliable, complete lifecycle+interaction event model for dynamic lists.** Every competitor either has no lifecycle events (Webflow, Framer, Bubble — interaction-only), or has them in code but not in the panel (Wix Studio Classic). By exposing `onItemReady`, `onItemUpdated`, and `onItemRemoved` in the Properties & Events panel alongside per-item interaction triggers — and generating item-scoped code stubs by default — Studio 2 would close the gap that the Velo group left open, and leapfrog what any competitor currently offers. The audience for this is Developers and Partners who today have to hand-write boilerplate or use workarounds to achieve what should be the default behavior.

## Recommended Scope

| Feature | Tier | Why |
|---------|------|-----|
| `onItemReady` in Properties & Events panel | Must-have | Table stakes — Velo already has it; must surface in Studio 2 panel |
| `onItemRemoved` in Properties & Events panel | Must-have | Completes the existing lifecycle pair |
| Per-item click/interaction handlers with item context auto-passed | Must-have | All competitors support click per item; Bubble's "Current cell's" model is the right reference |
| Hover/mouse events on dataset-connected Repeater (bug fix) | Must-have | Unique Wix regression; no competitor has this limitation; must fix before Studio 2 |
| `onItemUpdated` lifecycle event | Differentiator | No competitor has this; fills the #1 gap developers hit with dynamic data |
| `$item`-scoped code stubs generated by default | Differentiator | Mirrors Framer's "right default, overridable" model; eliminates the most common scoping mistake |
| Item-level state machine (Hover/Selected/Default states per Repeater item) | Not now | Framer's model is strong but complex to implement; revisit after lifecycle events are solid |
| No-code interaction composer for Repeater items | Not now | Webflow's visual interaction model for CMS lists still has gaps; don't replicate the debt |

## Suggested Next Steps

- **Competitive sign-up**: Try Webflow's Collection List hover interaction hands-on and Framer's code component event handler panel — both are free tier accessible and will sharpen the spec detail.
- **Internal discovery**: Confirm with the platform/Velo team which of these models are achievable given Studio 2's editor architecture — particularly `onItemUpdated` and `$item`-scoped code stub generation.
- **Open questions**: Does Studio 2's Properties & Events panel architecture support exposing lifecycle events (`onItemReady`, `onItemUpdated`, `onItemRemoved`) today, or does that require panel work? This gates the entire scope recommendation.

# Competitor Research — Repeater in Repeater (Studio 2 CMS)

**Project:** Repeater in Repeater (Studio 2 CMS)
**Phase:** Discovery — Competitor Research
**Date:** 2026-05-06
**Focus:** How comparable visual website builders / no-code platforms expose nested repeating lists (parent → child) tied to a CMS or database.

---

## Headline

**Nested repeating is table-stakes** in this category. Webflow and Framer ship it as a first-class CMS pattern. Bubble ships it too, but as a power-user pattern with performance gotchas. Squarespace does not have it — and that gap is consistently cited as a reason users outgrow Squarespace. Wix is currently grouped with Squarespace on this dimension; the Wix Blocks team has the engine working but it has not made it to the canvas yet. **There is no design risk in shipping this. The risk is shipping it slowly.**

---

## Competitor Summaries

### Webflow — gold standard
[Nest Collection Lists (Webflow Help Center)](https://help.webflow.com/hc/en-us/articles/33961268936851-Nest-Collection-lists) · [Nested collection lists are here (Webflow blog)](https://webflow.com/blog/nested-collection-lists-are-here) · [Webflow University video](https://university.webflow.com/videos/nest-collection-lists)

- **Shape:** Outer Collection List bound to a parent Collection. Drop a second Collection List inside the parent's item template; bind it to the child Collection. The child list automatically filters to items linked to the current parent via a multi-reference field.
- **Canvas affordance:** **Drag-and-drop.** The Designer recognizes the parent context and exposes the child's reference field as the natural binding source.
- **Depth:** **Up to 3 levels** (e.g., Course → Lesson → Assignment).
- **Limits:** Up to 10 nested Collection lists per page; up to 100 nested items per parent. Cannot nest inside components.
- **Workarounds for the limits:** A whole third-party ecosystem (Finsweet "List Nest," Philip Proth's open-source helper, jQuery patterns) exists to push past these limits. That itself is a signal — even with the limits, users want more nesting.
- **Why it works:** The mental model is the same as the platform's main primitive (Collection List). No new concept to learn. The reference-field-as-filter is silent — the user doesn't have to think about datasets.

### Framer — simple, but partial
[Framer Academy: Collection Lists](https://www.framer.com/academy/lessons/creating-cms-collection-lists) · [Framer Blog: Combining Multiple CMS Collections for Category Pages](https://www.framer.com/blog/category-pages/) · [Are nested CMS possible? (Community)](https://www.framer.community/c/support/are-nested-cms-possible)

- **Shape:** Collection Lists support filtering by another collection's slug, which lets you compose a category page that loads child items based on the URL parameter — that covers the *page-level* nested pattern.
- **Canvas affordance:** Insert panel → Collection Lists → drop in. Filtering uses slug-based lookup on dynamic pages.
- **Depth:** Effectively 2 levels via slug filtering on dynamic pages. **Not native nested rendering inline within a single list item.**
- **Limits:** Framer "doesn't currently support CMS variables for arrays" — limits stylistic control of nested children.
- **Tradeoff:** Cleaner UX than Bubble, but less expressive than Webflow. The "list inside a list inside one page" shape requires custom code or a workaround.

### Bubble — works, but power-user
[Repeating groups (Bubble Docs)](https://manual.bubble.io/help-guides/design/elements/web-app/containers/repeating-groups) · [Nested Repeating Groups (Bubble Forum)](https://forum.bubble.io/t/nested-repeating-groups/15783) · [Step-by-step guide (PlanetNoCode)](https://www.planetnocode.com/tutorial/nested-repeating-groups-in-bubble-a-step-by-step-guide)

- **Shape:** Place a Repeating Group inside another Repeating Group's cell. Inner group's data source is *"Current cell's [parent type]'s [child list field]."*
- **Canvas affordance:** Drag inside; manual data-source expression. Requires the user to mentally hold the parent-child reference graph.
- **Depth:** No hard limit, but multiple levels rapidly degrade.
- **Pain points users report:**
  - Accessing the parent cell's data from within the nested group is confusing — element tree structure can break the "current cell" reference.
  - **Performance:** Bubble's workload-unit pricing means nested groups quickly become expensive ("each nested search multiplies your database calls exponentially").
- **Tradeoff:** Maximum flexibility, minimum approachability. Self-Creators struggle.

### Squarespace — not really
[Summary blocks (Squarespace Help Center)](https://support.squarespace.com/hc/en-us/articles/206543337-Summary-blocks) · [Categories and tags](https://support.squarespace.com/hc/en-us/articles/205814438-Categories-and-tags) · [Can't filter by category on summary block (Forum)](https://forum.squarespace.com/topic/260106-cant-filter-by-category-on-summary-block/)

- **Shape:** Summary Block can pull from a Collection Page (blog, products, events) and **filter** by category or tag. There is no concept of binding a child collection inside a parent item.
- **Canvas affordance:** Summary block configuration only. No drag-into-item paradigm.
- **Depth:** Single level. Categories and tags are flat — no hierarchy.
- **Tradeoff:** Lowest learning curve in the category, but the lowest ceiling for structured-content sites. Users who need parent → child move to Squarespace alternatives, often Wix or Webflow.

### Wix Blocks (internal benchmark) — engine works
[Nest Dynamic Repeaters (Wix Blocks dev docs)](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/nest-dynamic-repeaters) · [Dynamic Repeaters in Blocks](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/dynamic-repeaters-in-blocks)

- **Shape:** Outer dynamic Repeater binds to parent collection. Inner widget receives the parent item via a prop; inside the widget the dev calls `include()` on a cross-collection field and reconnects the inner Repeater. Works.
- **Canvas affordance:** **None.** This is for app developers — code path only.
- **Depth:** Practical 2 levels.
- **Tradeoff:** Proves the runtime supports it. Does not reach Studio 2 canvas users at all. The Studio canvas needs to wrap this primitive in a no-code binding flow.

---

## Feature Comparison

| Capability | Webflow | Framer | Bubble | Squarespace | **Wix Studio 2 (today)** |
|---|---|---|---|---|---|
| Native nested-list rendering | ✅ Drag-into-item | Partial (slug-filter) | ✅ Repeating-in-repeating | ❌ | ❌ canvas / ✅ Blocks code |
| Depth limit | 3 levels | ~2 (page-level) | Unbounded (perf-bound) | n/a | Unbounded (Blocks code only) |
| Discoverability for Self-Creator | High (drag) | Medium | Low (data-source expression) | n/a | Low (no canvas affordance) |
| Reference-field-as-filter is implicit | ✅ | Partial | ❌ user wires | n/a | ❌ user wires (multi-dataset hack) |
| Code-free | ✅ | ✅ | ✅ | n/a | ❌ in Studio 2 |
| Performance guardrails | Hard caps (10 lists, 100 items) | Loose | None — user pays | n/a | n/a |
| Per-instance independent filters within nested | ✅ | Partial | ✅ | n/a | ❌ (UV: 76 mentions on shared-dataset filter) |

---

## Patterns Across Competitors

1. **Drag-into-item is the winning canvas idiom.** Webflow's "drop a Collection List inside the parent's template" maps directly to user mental models. Bubble forces a data-source expression — users complain. Wix Studio should mimic Webflow's drag-and-drop, surfacing the parent's array/multi-reference fields automatically as the binding source for the inner Repeater.
2. **A depth limit of 2–3 levels covers virtually all use cases.** Webflow ships 3, says it covers Course → Lesson → Assignment. Real-world content rarely needs more. Wix v1 should pick **2** and call it out clearly; users at 3+ are an edge case better served by code/Blocks.
3. **Implicit reference-field-as-filter is the magic.** When the inner list is bound to "items linked to current parent," the user doesn't need to think about datasets. This is the single biggest UX delta vs. the Wix multi-dataset workaround.
4. **Hard caps on items-per-parent prevent runaway pages.** Webflow's 100-item cap is a smart guardrail. Wix should set a similar cap and surface it in the inspector.
5. **Performance footguns are real.** Bubble's "every nested search multiplies database calls" is a cautionary tale. Wix's runtime should batch the cross-collection fetch (Blocks `include()` already does this) — never let the canvas flow result in N+1 queries.

---

## Opportunity Areas for Wix Studio 2

1. **Match Webflow's drag-into-item paradigm with `This item` scope in the binding picker.** When the inner Repeater's `Items` is opened and the cursor is inside another Repeater's item, the binding panel should auto-suggest the parent item's array / multi-reference fields — labeled `This item`, `This repeater`, page-level — in that order. This mirrors the existing `This repeater` work in the blank-Repeater PRD.
2. **Pick a depth of 2 for v1, document it.** Avoid Bubble's "unbounded but slow" trap.
3. **Reuse the Blocks runtime primitive.** Don't build a parallel data path for the canvas. Wrap `include()` in the editor's binding platform.
4. **Cap nested items per parent.** Surface the cap in the inspector with a clear message — Webflow's 100 is a reasonable starting point.
5. **Solve the "independent filters" complaint at the same time.** The 76-mention shared-dataset complaint is partly the same gap — users want per-instance state for the inner Repeater. Bake this into the nested model.
6. **Lead with template-level wins.** Stores categories→products, Restaurants menus→dishes, Bookings services→staff, Blog categories→posts. Ship templates that demonstrate the new primitive — that's what unlocks the "magical first 60 seconds" Webflow built its reputation on.

---

## Risks and Open Questions

- **Depth ceiling.** 2 vs. 3 — confirm with engineering and with the data query Q4 (depth distribution among workaround attempts).
- **Components / sections.** Webflow does *not* allow nested Collection Lists inside components. Should Studio 2 sections / global blocks have the same restriction? Probably yes for v1.
- **Mismatch case.** What if the inner Repeater is also given an independently attached context that does not match the parent's array? Inspector must make the mismatch visible (carry-over from `blank-repeater-prd.md` section 10 — extend explicitly to nested).
- **Existing-workaround migration.** Should sites that already use multi-dataset hacks be auto-migrated to the new primitive, or get a "convert" affordance in the inspector? Worth a follow-up decision.

---

## Sources

- [Webflow — Nest Collection Lists](https://help.webflow.com/hc/en-us/articles/33961268936851-Nest-Collection-lists)
- [Webflow Blog — Nested Collection Lists Are Here](https://webflow.com/blog/nested-collection-lists-are-here)
- [Webflow University — Nest Collection Lists video](https://university.webflow.com/videos/nest-collection-lists)
- [Finsweet List Nest attribute](https://finsweet.com/attributes/list-nest)
- [Multiple Nested Collection Lists — Webflow Forum](https://discourse.webflow.com/t/multiple-nested-collection-lists/125123)
- [Framer Academy — Collection Lists](https://www.framer.com/academy/lessons/creating-cms-collection-lists)
- [Framer Blog — Combining Multiple CMS Collections for Category Pages](https://www.framer.com/blog/category-pages/)
- [Framer Community — Are nested CMS possible?](https://www.framer.community/c/support/are-nested-cms-possible)
- [Bubble Docs — Repeating groups](https://manual.bubble.io/help-guides/design/elements/web-app/containers/repeating-groups)
- [Bubble Forum — Nested Repeating Groups](https://forum.bubble.io/t/nested-repeating-groups/15783)
- [Avalan Labs — How to Build Nested Repeating Groups in Bubble](https://www.avalanlabs.co/blog/how-to-build-nested-repeating-groups-in-bubble-io)
- [Squarespace — Summary blocks](https://support.squarespace.com/hc/en-us/articles/206543337-Summary-blocks)
- [Squarespace — Categories and tags](https://support.squarespace.com/hc/en-us/articles/205814438-Categories-and-tags)
- [Wix Blocks — Nest Dynamic Repeaters](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/nest-dynamic-repeaters)
- [Wix Blocks — Dynamic Repeaters in Blocks](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/dynamic-repeaters-in-blocks)

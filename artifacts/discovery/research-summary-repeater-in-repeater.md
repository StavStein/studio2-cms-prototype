# Research Summary — Repeater in Repeater (Studio 2 CMS)

**Date:** 2026-05-06
**Project:** Repeater in Repeater (Studio 2 CMS)
**Goal:** Define a full PRD for nested Repeaters in Studio 2 — a Repeater placed inside another Repeater's item template, bound to the parent item's array / multi-reference field.
**Research phase:** Discovery complete (pre-spec).

---

## 1. Research Overview

**Problem space.** Studio 2 today binds a Repeater to one collection at a time, flat. Builders who need parent → child shapes (categories with products, seasons with episodes, meals with dishes, courses with lessons, properties with contacts) cannot express that natively — they fall back on multi-dataset hacks, side-by-side filtered Repeaters, or Velo / Wix Blocks code. The runtime can do nested already (Wix Blocks ships it via `include()` + widget prop-drilling), but the canvas binding model has not exposed it. This research answers: how big is the gap, how do competitors solve it, what already exists internally, and what should we call things in the PRD.

**Research conducted:**

| Research Type | Source ID | Status |
|---|---|---|
| Project Brief | SRC-BRIEF | ✅ [`project-brief-repeater-in-repeater.md`](../project-brief-repeater-in-repeater.md) |
| User Voice Research | SRC-UV | ✅ [`user-voice-repeater-in-repeater.md`](user-voice-repeater-in-repeater.md) (User Voice MCP, web; Jira auth unavailable — flagged) |
| Data Research Plan | SRC-DATA | ✅ [`data-analysis-plan-repeater-in-repeater.md`](data-analysis-plan-repeater-in-repeater.md) |
| Data Analysis | SRC-DATA-ANALYSIS | ⚠️ [`data-analysis-repeater-in-repeater.md`](data-analysis-repeater-in-repeater.md) — not executed (OpenMetadata search tool 404 in this environment); plan documents queries for follow-up |
| Competitor Research | SRC-COMP | ✅ [`competitor-research-repeater-in-repeater.md`](competitor-research-repeater-in-repeater.md) (Webflow, Framer, Bubble, Squarespace, Wix Blocks) |
| Wix Internal Research | SRC-INT | ✅ [`internal-discovery-repeater-in-repeater.md`](internal-discovery-repeater-in-repeater.md) |
| Terminology Research | SRC-TERM | ✅ [`terminology-research-repeater-in-repeater.md`](terminology-research-repeater-in-repeater.md) (Wix UX Writing Glossary API not queried — flagged) |

**Target audience.** Studio Users (primary), Self-Creators on Studio 2 (secondary), Partners / agencies. End-users of the resulting sites consume the parent → child shape (categories with products, etc.).

---

## 2. Research Summaries

### 2.1 User Voice (SRC-UV)

**Strong, high-confidence signal.** Triangulated across support tickets, Wix Help Center, Wix Studio Community Forum, and Velo Forum.

- **The dominant pain shape is "list of children per parent."** Users describe it in domain terms — *speaker's courses*, *plant's planets*, *meal's dishes*, *related packages* — not as "nested repeater."
- **Three internally-labeled support clusters confirm scale:**
  - Multi-reference field not displaying dynamic data in repeater — **108 highlights**
  - Repeaters cannot have independent filters due to shared dataset — **76 highlights**
  - Inconsistent spacing in repeater elements (subset describing "multiple repeaters instead of a unified structure") — within 119 highlights
- **The platform partly admits it.** Wix Help Center has a literal request page titled *"CMS Request: Attaching a Repeater onto Another Repeater"* — *"would allow you to group items within a repeater based on a reference field. For example, dishes within the main repeater by meal."* The Wix Studio Forum has 4+ active threads.
- **Wix Blocks already ships nested dynamic Repeaters** for app developers via `include()` + widget prop-drilling — so the engine works, the canvas just doesn't expose it.

### 2.2 Competitor (SRC-COMP)

**Nested repeating is table-stakes.** No design risk; the risk is shipping slowly.

- **Webflow** — gold standard. Drag a Collection List into the parent's item template; binds via multi-reference. Up to **3 levels deep**; caps at 10 nested lists per page and 100 nested items per parent. The reference-field-as-filter is implicit — magical.
- **Framer** — partial. Slug-based filtering on dynamic pages covers the page-level pattern, but inline list-in-list inside a single page needs custom code.
- **Bubble** — works but power-user. "Current cell's [parent type]'s [child list field]" expression. Performance footgun: every nested search multiplies DB calls.
- **Squarespace** — does not have it. Categories/tags only filter Summary Blocks; users outgrow Squarespace specifically for this.
- **Wix Blocks (internal benchmark)** — `include()` + widget prop-drilling. Code-only. Engine confirmed.

**Patterns:** drag-into-item is the winning idiom; depth 2–3 covers virtually all use cases; implicit reference-field-as-filter is the magic; hard caps prevent runaway pages; performance footguns are real.

### 2.3 Data Research Plan (SRC-DATA)

Planned queries grouped into Go/No-Go and How-to-Build. Highest-priority three:

- **Q1.** Count Studio 2 sites attempting any of the three workaround patterns in the trailing 90 days. Slice by segment.
- **Q2.** Customer Care volume attributable to the three workaround clusters per quarter, trailing 12 months.
- **Q3.** Publish funnel for first-time workaround sites (60-day publish, 30-day return-to-edit) vs. matched control.

**Go/No-Go gate:** Q1 ≥ 5% AND Q2 cluster volume ≥ 5% of CMS-related CC volume → green-light. Build decision is largely qualitatively supported already; data calibrates scope, not bet/no-bet.

### 2.4 Data Analysis (SRC-DATA-ANALYSIS)

Not executed. Trino + KB-retrieval reachable; OpenMetadata table-search returned 404 in this environment, blocking table discovery. Plan documents the queries for a data-team-led follow-up.

### 2.5 Wix Internal (SRC-INT)

- The runtime supports nesting via `wixData.queryReferenced()` / `include()` (Blocks). **Reuse, don't rebuild.**
- Pagination already branches: `renderSettingsRepeaterOnRepeater()` is a precedent for nested-specific inspector logic. Extend that branch to binding/disconnect/replace.
- Multi-reference field rendering is the data plumbing under nested. Fixing nested without addressing this leaves users stuck.
- The blank-Repeater PRD is the structural model — same depth, same patterns (initial state, connect, replace, disconnect, edge cases).
- Open dependencies: depth limit decision (recommend 2 for v1), mobile parity, component / global block restriction (Webflow blocks nested Lists inside components — recommend Studio 2 follow suit).

### 2.6 Terminology (SRC-TERM)

- **Recommended object terms:** Nested Repeater, Outer Repeater, Parent item, Child item, Child array / Child collection, Parent row context.
- **Recommended scope badge:** **`This item`** — sibling to existing `This repeater`. Goes top of the binding picker when the Repeater is nested.
- **Recommended depth limit:** 2 for v1.
- **Avoid:** "Sub-Repeater," "Inner Repeater," "Child Repeater," "Collection List," "Repeating Group."
- **Pending manual check:** Wix UX Writing Glossary API (VPN required) — not queried in this pass; confirm new terms don't collide with prohibited terms in the Studio Editor or Wix CMS domain.

---

## 3. Key Observations

| ID | Observation | Source |
|---|---|---|
| OBS-1 | The "list of children per parent" shape is the dominant content-site pattern, not a niche power-user request | SRC-UV, SRC-COMP |
| OBS-2 | Three documented workarounds — multi-dataset filter, side-by-side filtered Repeaters, Velo/Blocks code — all break in predictable ways the support team already labels | SRC-UV, SRC-INT |
| OBS-3 | Wix Help Center has a dedicated *"CMS Request: Attaching a Repeater onto Another Repeater"* page — Wix has acknowledged the gap in its own framing | SRC-UV |
| OBS-4 | The Wix Blocks team already ships nested dynamic Repeaters via `include()` + widget prop-drilling — the runtime primitive exists | SRC-COMP, SRC-INT |
| OBS-5 | The Studio 2 prototype already branches the inspector for nested Repeaters via `renderSettingsRepeaterOnRepeater()`, but only for pagination | SRC-INT |
| OBS-6 | Webflow's drag-into-item paradigm with implicit reference-field-as-filter is the gold-standard UX; Squarespace lacks the capability entirely; Bubble has it but with performance footguns | SRC-COMP |
| OBS-7 | A binding-picker scope badge model already exists (`This repeater`, page-level); adding `This item` is a sibling extension, not a new mechanism | SRC-TERM, SRC-INT |
| OBS-8 | Multi-reference fields are the dominant data shape behind nested Repeaters; fixing nested without addressing the existing 108-mention multi-reference rendering cluster leaves users stuck | SRC-UV, SRC-INT |
| OBS-9 | Realistic depth ceiling for v1 is 2 levels; Webflow ships 3, but most documented use cases (course→lesson, category→product, meal→dish) fit 2 | SRC-COMP, SRC-INT |
| OBS-10 | Bubble's nested-list performance horror story (every nest multiplies DB calls) is the cautionary signal — Studio 2 must batch via `include()`/`queryReferenced()` to avoid N+1 queries | SRC-COMP |

---

## 4. Insights

### Insight 1 — Nested Repeater is a coverage gap, not a discovery gap
**Insight:** Wix knows the gap exists, has documented it on the Help Center, and has a working runtime via Blocks. The work is bringing it to the Studio canvas — not designing a new mechanism.
**Evidence:** OBS-3, OBS-4, OBS-5
**So what:** The PRD can lean on existing artifacts (blank-Repeater PRD, pagination spec, Blocks runtime). The risk profile is "adapt-and-extend," not "invent-and-validate."

### Insight 2 — The dominant data shape (multi-reference) is already in pain
**Insight:** The 108-highlight multi-reference rendering cluster is the data plumbing under nested Repeaters. Shipping nested without addressing the multi-reference rendering pain leaves users stuck at the field level, even if the Repeater binds correctly.
**Evidence:** OBS-2, OBS-8 (SRC-UV)
**So what:** PRD scope must include surfacing parent-item array / multi-reference fields cleanly in the binding picker. A nested Repeater with a working binding flow but broken field surfacing is half a feature.

### Insight 3 — Drag-into-item with `This item` scope is the right canvas idiom
**Insight:** Webflow's UX is the proven pattern: drop the inner Repeater into the parent's item, the binding picker auto-suggests the parent's array fields. Wix already has the binding-picker scope-badge mechanism (`This repeater`); adding `This item` is a sibling, not a new concept.
**Evidence:** OBS-6, OBS-7
**So what:** The PRD specifies the binding picker should add `This item` as a third scope, ranked above `This repeater` and page-level when the Repeater is nested. The drag-into-item gesture should auto-bind to the first available parent-item array field (carry-over from blank-Repeater PRD §5).

### Insight 4 — Depth = 2 for v1, with a clear cap and surface
**Insight:** Webflow's 3-level cap is the ceiling; 2 covers virtually all real use cases (category→product, meal→dish, season→episode, course→lesson). Bubble's "unbounded but slow" trap is the warning.
**Evidence:** OBS-9, OBS-10
**So what:** v1 ships 2 levels with a hard cap, surfaced in the inspector. Items-per-parent should mirror Webflow's 100 unless engineering data argues otherwise.

### Insight 5 — Disconnect cascades and replace warnings extend, not invent
**Insight:** Blank-Repeater PRD already specifies replace-context warnings and disconnect cascade copy. Nested adds one new dimension: when the **outer** Repeater's `Items` is changed or removed, the inner Repeater's binding cascades. Same warning pattern, deeper cascade count.
**Evidence:** OBS-5, blank-Repeater PRD
**So what:** Reuse copy patterns from blank-Repeater PRD §7, §8. Update count must include grandchild bindings. No new mechanism needed.

### Insight 6 — The terminology spine is `Nested Repeater` / `Outer Repeater` / `Parent item` / `This item`
**Insight:** "Sub-Repeater," "Child Repeater," "Inner Repeater," and "Repeater within Repeater" are all in user vocabulary but ambiguous or overloaded. "Nested Repeater" is unambiguous; "Outer Repeater" reads cleaner than "Parent Repeater" because the items themselves are also called parent rows.
**Evidence:** SRC-TERM
**So what:** Adopt `Nested Repeater` / `Outer Repeater` / `Parent item` / `Child item` / `This item` (scope badge) consistently across PRD, inspector, panel copy, dev docs.

---

## 5. Confidence Assessment

| Insight | Confidence | Reasoning |
|---|---|---|
| Insight 1 (coverage gap, not discovery) | **High** | Multiple Wix-owned surfaces confirm. Runtime proven via Blocks. |
| Insight 2 (multi-reference is the data shape) | **High** | 108-highlight cluster + Help Center docs + competitor patterns all align. |
| Insight 3 (drag-into-item + `This item`) | **High** | Webflow's pattern is established; sibling scope badge already exists in Wix. |
| Insight 4 (depth = 2 v1) | **Medium-high** | Recommended based on use cases, but engineering input on runtime guarantees outstanding. Q4 in data plan would calibrate. |
| Insight 5 (extend disconnect/replace) | **High** | Pattern already specified; mechanical extension. |
| Insight 6 (terminology spine) | **Medium-high** | Internally consistent and tested against competitor terms. Pending Wix UX Writing Glossary cross-check. |

---

## 6. Open Questions for Spec

1. **Depth ceiling.** 2 vs. 3 for v1. Recommend 2; revisit if data Q4 shows meaningful 3-level use.
2. **Items-per-parent cap.** 100 (Webflow parity) vs. tighter / looser. Engineering input needed.
3. **Mobile parity.** Whether mobile Repeater shares the nested binding path or needs its own spec.
4. **Global sections restriction.** Should a Nested Repeater placed inside a Studio 2 **global section** (a reusable block shared across pages) support the `This item` binding flow? Recommendation: **no, not in v1** — match Webflow's rule that nested Collection Lists cannot live inside Components. A global section has no stable parent data context (it can land on any page, inside any Outer Repeater or none), so propagating the parent row context into it reliably is out of scope for v1. In practice: the `This item` scope should not appear in the binding picker when the Repeater is inside a global section, and the inspector should surface a clear explanation. Confirm scope and copy in PRD review.
5. **Existing-workaround migration.** Auto-migrate, offer "convert" affordance, or leave alone? Worth a follow-up decision after data Q1 sizes the affected base.
6. **AI flows.** Defer to a separate AI section as the blank-Repeater PRD does.
7. **`onItemUpdated` interplay.** The repeater-event-handlers research flagged a missing lifecycle event; nested Repeaters compound the need. Coordinate the two PRDs.
8. **Glossary cross-check.** Confirm `Nested Repeater`, `Outer Repeater`, `Parent item`, `Child item`, `This item` against the Wix UX Writing Glossary (VPN required).

---

## 7. Recommended Next Steps

1. **Draft the PRD** (`ck-product-spec`) using the blank-Repeater PRD as the structural model. Include sections for: initial state of a Nested Repeater, attaching/inheriting parent context, binding to a Child array via the `Items` picker with `This item` scope, replace-context cascade, disconnect-Items cascade, depth limit, items-per-parent cap, context mismatch handling, mobile parity, AI scope deferral.
2. **Run the data plan** (`ck-data-query` follow-up by the data team) once OpenMetadata table-search is restored — Q1, Q2, Q3 first to size the bet and prioritize verticals.
3. **Engineering alignment session** with Blocks team and runtime engineering on `include()` / `queryReferenced()` reuse and depth/cap guarantees.
4. **Glossary cross-check** for the new terms.
5. **Re-run Jira** (`UPI`, `EDITOR`, `CMS`, `STUDIO`) once Jira auth is restored to capture any prior ticketed discussion.

---

## 8. Sources

- [Project Brief](../project-brief-repeater-in-repeater.md) — SRC-BRIEF
- [User Voice](user-voice-repeater-in-repeater.md) — SRC-UV
- [Data Research Plan](data-analysis-plan-repeater-in-repeater.md) — SRC-DATA
- [Data Analysis status](data-analysis-repeater-in-repeater.md) — SRC-DATA-ANALYSIS
- [Competitor Research](competitor-research-repeater-in-repeater.md) — SRC-COMP
- [Wix Internal Discovery](internal-discovery-repeater-in-repeater.md) — SRC-INT
- [Terminology Research](terminology-research-repeater-in-repeater.md) — SRC-TERM
- [Blank Repeater PRD](../../blank-repeater-prd.md) (existing artifact, structural model)
- [Pagination Spec](../product/product-spec-repeater-pagination.md) (existing artifact, nested-precedent)
- [Repeater Context Meeting Checklist](../../repeater-context-meeting-checklist.md) (existing artifact)
- [Binding Platform Demo HTML](../../pages/context-binding-prototype/binding-platform-demo.html) (working prototype)

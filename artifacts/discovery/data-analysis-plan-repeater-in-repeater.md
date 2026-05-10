# Data Analysis Plan — Repeater in Repeater (Studio 2 CMS)

**Project:** Repeater in Repeater (Studio 2 CMS)
**Phase:** Discovery — Data Analysis Plan
**Date:** 2026-05-06
**Inputs:** project brief, user voice research. Competitor research and analytical KB lookup not yet run.

---

## 1. Title

Nested Repeater (Repeater inside a Repeater item) for Studio 2 CMS.

## 2. Coverage Assessment

| Dimension | Coverage | Notes |
|---|---|---|
| Target audience | **Strong** | Studio Users, Self-Creators, Partners — confirmed in brief and UV |
| Problem & need validation | **Strong** | UV triangulated across support, Help Center request page, Studio Forum, Velo forum |
| Business impact & KPIs | **Partial** | Workaround load is named in support data; needs sizing |
| Adoption & usability | **Partial** | Needs prototype-driven measurement |
| Strategic alignment | **Strong** | Studio Editor + Wix Data (CMS) gameplans |
| Ecosystem & cross-domain | **Partial** | Touches CMS, Stores (categories→products), Restaurants (meals→dishes), Bookings (services→staff), Blog (categories→posts) |
| Competitive landscape | **Not yet researched** | Webflow / Framer / Bubble / Squarespace expected to support nested |

## 3. Problem Statement

Builders who organize content as parent→children (categories with products, seasons with episodes, meals with dishes, courses with lessons, properties with contacts) cannot render that shape natively in Studio 2. The current Repeater binds to one collection at a time; child arrays of the parent item are not first-class binding sources. Users fall back to three documented hacks — multi-dataset filter chaining, page duplication with hardcoded filters, or Velo / Wix Blocks code with `include()` and prop-drilling. All three break in predictable ways the Customer Care team is already labeling internally ("multiple repeaters instead of a unified structure"). Nested repeating is the dominant content-site shape, not a niche power feature, and the gap costs Self-Creators the most because the workarounds all require leaving the canvas.

## 4. Target Audience

**Primary:** Studio Users (agencies, freelancers, advanced creators on Studio 2) building data-rich client sites — they hit the gap first because their projects involve catalogs, directories, and structured content.

**Secondary:** Self-Creators on Studio 2 who manage their own catalog / listing / membership / events / restaurants sites. They are the ones least equipped to use the Velo / Blocks workaround and most likely to abandon or open a support ticket.

**User-of-User:** Site visitors — the parent→child shape is what end users expect to see (categories with products, seasons with episodes). Bad workarounds = bad page experience.

## 5. Expected Business Outcome

- **Workaround load reduction:** Decrease in Customer Care tickets/calls labeled with the three workaround clusters (multi-reference-in-repeater, shared-dataset-filter, multiple-repeaters-spacing).
- **Activation:** Increase in % of Studio 2 sites with 2+ Repeaters that bind to a parent→child structure (vs. faking it with side-by-side filtered Repeaters on the same page).
- **Retention:** Reduce churn signal from sites that try CMS + Repeater and abandon (proxy: site published without any data-bound Repeater after the user attached one in the editor).
- **Template breadth:** Unlock template categories that are awkward today (catalog with sub-collections, restaurants, learning, real estate listings, member directories).

## 6. Domain Context

**Studio Editor** (KB id `b4105bf2-66c7-4a75-8934-5c4ab23ebc19`) — the surface this PRD lives on. Studio's strategic priority is professional-builder fluency for agencies and freelancers; data-driven layouts are core.

**Wix Data / CMS** (KB id `781cbc83-58f3-4ea3-b436-fe81b2f799d6`) — the data substrate. Multi-reference fields and reference fields are the data shape that maps cleanly to nested binding. Listed competitors include Squarespace, Bubble, Webflow, Sanity, Adobe — most of which expose nested-list rendering natively.

**Help Center signal:** A dedicated request page exists titled *"CMS Request: Attaching a Repeater onto Another Repeater"* — official acknowledgement that the platform doesn't support what users want.

**Wix Blocks signal:** Already supports nested dynamic Repeaters via `include()` + widget prop-drilling. The runtime can do it. Studio canvas just doesn't expose it.

## 7. Competitive Landscape

Not researched in this pass. Expected next step (`ck-research-competitors`) should cover Webflow Collection Lists nested in Collection Lists, Framer CMS, Bubble repeating groups, Squarespace, Sanity. Initial assumption from public Wix Studio Forum threads: nested is **table-stakes** in this category, not a differentiator. Validate before investing.

## 8. Key Decisions to Make

| Decision | Priority |
|---|---|
| Bind inner Repeater's `Items` from parent item's array fields with a `This item` scope badge | **Must-resolve** |
| Depth limit (recommend 2 levels for v1, surface clearly) | **Must-resolve** |
| Disconnect cascade behavior when parent's `Items` changes / is removed | **Must-resolve** |
| Whether the inner Repeater can also attach an independent context (mismatch case) | **Should-resolve** |
| Pagination semantics inside a nested context — re-use `renderSettingsRepeaterOnRepeater()` path? | **Should-resolve** |
| AI-assisted nested binding (defer to a separate AI section, like blank-Repeater PRD does) | **Nice-to-know** |
| Runtime backed by existing Blocks `include()` primitive vs. a new path | **Should-resolve** (engineering input) |

## 9. Questions to Investigate

### Go / No-Go (validate the bet)

**Q1. How many Studio 2 sites already attempt the nested shape via workarounds?**
- **Population:** Sites on Studio 2 with at least one Repeater bound to a CMS collection.
- **Cohort filter:** Sites where the page contains 2+ Repeaters bound to the same dataset OR Repeater bound to a multi-reference field via secondary dataset OR Repeater inside a dynamic page filtered by a reference field.
- **Timeframe:** Last 90 days (rolling).
- **Logic:** Count distinct sites matching any of the three workaround patterns. Slice by editor surface (Studio 2 vs Editor).
- **KB reference:** `domains-kb-studio-editor`, `domains-kb-wix-data-(cms)` (analytical KB not yet queried).

**Q2. What share of Customer Care contact volume is attributable to the three workaround clusters?**
- **Population:** Customer Care tickets, chats, and phone calls in the last 12 months tagged to Wix Data (CMS) or Studio products.
- **Cohort filter:** Tickets matching the three User Voice issue clusters: `multi-reference field not displaying dynamic data in repeater` (108), `repeaters cannot have independent filters due to shared dataset` (76), `inconsistent spacing in repeater elements` filtered to descriptions mentioning "multiple repeaters" (subset of 119).
- **Timeframe:** Trailing 12 months, broken into rolling quarters.
- **Logic:** Sum highlight count and unique reporters per cluster per quarter; trend direction.
- **KB reference:** UserVoice MCP labeled-issue store.

**Q3. Do sites that attempt the workaround actually publish?**
- **Population:** Sites that hit any workaround pattern in Q1 in the last 180 days.
- **Cohort filter:** First-time attempt of the workaround pattern (the user did not have it before).
- **Timeframe:** 60-day forward window from first-attempt date.
- **Logic:** Compare publish rate, time-to-first-publish, and 30-day return-to-edit rate vs. matched control of CMS-Repeater sites that don't hit the pattern.
- **KB reference:** Studio editor KB (publish + retention KPIs).

**Go/No-Go gate:** If Q1 shows <2% of CMS-Repeater Studio 2 sites attempting the shape OR Q2 shows the three clusters are <3% of CMS-related CC volume, deprioritize. If both are ≥5%, fast-forward to spec.

### How to Build It (calibrate the design)

**Q4. Among workaround attempts, what depth do users actually need?**
- **Population:** Sites in Q1's workaround cohort.
- **Cohort filter:** Page-level — count distinct nesting depths attempted (1-level child, 2-level grandchild, 3+).
- **Timeframe:** Last 90 days.
- **Logic:** Distribution of depth per page; flag pages with 3+ levels separately. Cross-tab with vertical (Stores, Restaurants, Bookings, Blog, generic CMS).
- **KB reference:** Wix Data (CMS) field-type usage tables.

**Q5. Which parent collection types drive the most nested attempts?**
- **Population:** Q1 workaround cohort.
- **Cohort filter:** Parent collection's source: native Wix Stores categories, Wix Restaurants menus, Wix Bookings services, Wix Blog categories, generic Wix Data collection.
- **Timeframe:** Last 90 days.
- **Logic:** Distribution across source types; flag the top 3 — they are the templates that should ship first.
- **KB reference:** CMS analytical tables (collection-type usage).

**Q6. Where in the site structure does nested Repeater appear?**
- **Population:** Q1 workaround cohort.
- **Cohort filter:** Page type — dynamic item page, dynamic list page, regular page, member page.
- **Timeframe:** Last 90 days.
- **Logic:** Distribution across page types; check whether the nested intent skews to dynamic item pages (which would change the binding-picker default).
- **KB reference:** Studio editor KB (page-type taxonomy).

**Q7. What share of nested attempts are driven by Velo / Blocks code vs. canvas-only?**
- **Population:** All sites attempting the nested shape in the last 90 days.
- **Cohort filter:** Site contains any Velo file referencing `$w('#repeater').onItemReady` with nested `setData` OR site uses a Blocks widget with nested Repeater.
- **Timeframe:** Last 90 days.
- **Logic:** % code-path vs. canvas-path; cross-tab by user segment (Self-Creator / Studio User / Partner / Developer).
- **KB reference:** Velo + Blocks usage tables.

**Q8. Do users abandon the workaround?**
- **Population:** Sites that started a workaround attempt in the last 180 days.
- **Cohort filter:** First-attempt site sessions where the workaround was added.
- **Timeframe:** 14-day forward window.
- **Logic:** % of sessions that remove the workaround within 14 days; % that publish anyway with the workaround in place; % that submit a CC ticket within the window.
- **KB reference:** Studio editor session-event tables.

## 10. Open Gaps

- **Sizing of intent vs. workaround.** We know who hits the workaround. We don't yet know who *wanted* it but never tried — those users churned silently or stayed flat. Gap addressed only via UV deep-dive (already done) and competitor switching-intent searches.
- **Help Center request page interaction data.** If the request page tracks signups / votes / "I want this too" clicks, that's a direct demand signal. Not pulled in this plan.
- **Partner Forum and Wix Wishlist** — auth-gated; recommended manual scrape for thread / vote counts.

## 11. KB-Anchored Context

Analytical KB was not queried in this pass (skill rule: 1 Navigator + 1 domain max; both used in the project brief). Recommended follow-up:

- **Studio editor KB** — pull tables for Repeater usage, CMS-bound Repeater usage, publish funnel from CMS-Repeater attempt, sessions-with-Velo flag.
- **Wix Data (CMS) KB** — pull tables for multi-reference field usage, secondary-dataset count per page, dataset filter cascading patterns.
- **Customer Care KB** — issue-level highlight counts, trend per quarter, reporter uniqueness.

Until those are pulled, the metric KB references in section 9 are pointers, not table names.

## 12. Go/No-Go Gate & Dependencies

**Gate:** Q1 ≥ 5% of CMS-Repeater Studio 2 sites attempting any workaround pattern AND Q2 cluster volume ≥ 5% of CMS-related CC volume → green-light. Either signal alone ≥ 5% with the other ≥ 3% → green-light with scope reduction. Both <3% → deprioritize.

**Dependencies:**
- Q3 depends on Q1 (need cohort).
- Q4–Q7 depend on Q1 (need cohort).
- Q5 informs which template categories ship first.
- Q6 informs the binding-picker default scope (page-level vs. parent-item).
- Q8 informs whether the new flow needs an explicit migration / cleanup path for existing workarounds.

---

## Recommended Next Skills

1. **`ck-research-competitors`** — confirm whether nested Repeaters are table-stakes or a differentiator across Webflow / Framer / Bubble / Squarespace / Sanity. Highest-leverage missing piece.
2. **`ck-data-query`** — actually run Q1, Q2, Q3 against analytical tables once KB IDs are mapped.
3. **`ck-research-wix-internal`** — surface prior Wix discussions, Jira tickets, and Blocks team learnings.

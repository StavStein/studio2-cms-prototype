# Repeater in Repeater — Vertical Data Investigation Research Brief

---

**Coverage assessment:**
- Problem statement: **Strong** — build committed at spec Draft v5; problem fully validated by UV research and 155+ support tickets
- Target audience: **Strong** — Studio Users, Self-Creators, Partners confirmed across UV, spec, and project brief
- Expected business outcome: **Partial** — workaround reduction, activation, retention named; not yet sized to real cardinality data
- Competitive landscape: **Researched** — Webflow (depth 3, 100-item hard cap), Framer, Bubble, Squarespace all documented
- Dimensions covered: 5 of 7 — Target Audience, Problem validation, Strategic alignment, Ecosystem, Competitive landscape
- Dimensions not fully covered: Business impact quantification (sizing), Adoption/usability against real vertical data
- Recommended before spec finalization: run Q1 and Q5 to close OQ-1 (warning threshold) before engineering commits to FR-017 threshold

---

## Problem Statement

The Repeater in Repeater spec is at Draft v5 with a full functional requirements set, engineering delta tags, and a complete open-questions log. The build decision is made. However, three quantitative parameters — the default nested item limit (10 per parent row), the multiplicative performance warning threshold (200 combined items), and the supported depth limit (3 levels) — were set as calibrated starting points, not derived from real usage data. The spec explicitly flags OQ-1 as an open question: *"calibrate the threshold against real benchmark data."* OQ-3 defers audience sizing to a future data pass.

This investigation asks: do the spec's numbers reflect what Wix merchants actually encounter in their data? Three verticals are the primary test cases: Wix Stores (Category → Products, the canonical RiR use case named in the spec), Wix Events (Event Category → Events on a listing page, or Event → Sessions), and Wix Bookings (Service Category → Services). Without real cardinality data, default=10 may be too conservative (forcing most users to change it immediately) or too permissive. The 200-item warning threshold could fire too early for typical Stores use (noise for median users) or too late (allowing users to build pages that genuinely degrade performance). Depth-3 support requires real 3-level hierarchies to exist in Wix data to justify the implementation cost over depth-2.

## Target Audience

- **Primary user:** Studio Users (agencies, advanced creators on Studio 2) and Self-Creators building catalog, listing, or service sites
- **User-of-User:** Site visitors who experience the rendered nested content (buyers browsing categorized products, attendees browsing event listings, clients browsing service menus)
- **Target segment for this investigation:** Active Stores sites (≥1 order in last 90 days) as the primary data source; active Events and Bookings sites as secondary verticals
- **Other segments:** Partners building structured-content client sites; Self-Creators on Restaurants, Blog, and generic CMS collections are out of scope for this pass but noted as downstream use cases in the existing data plan

## Expected Business Outcome

Accurate calibration of the three spec parameters reduces friction at two touchpoints: during canvas setup (the warning threshold should fire only when performance genuinely matters, not as noise for the median use case), and at first use (the default of 10 should cover the median user without forcing an immediate change). An over-conservative default creates a "why is my data cut off?" support ticket pattern; an over-permissive threshold creates slow-page complaints post-publish. This investigation directly closes OQ-1 and provides the audience size for OQ-3, enabling the spec to move to final review with grounded quantitative parameters.

## Domain Context

Studio Editor (KB id `b4105bf2-66c7-4a75-8934-5c4ab23ebc19`) + Wix Data / CMS (KB id `781cbc83-58f3-4ea3-b436-fe81b2f799d6`). No analytical KB was available in this pass — KB MCP was unreachable. Context below is from the existing RiR project brief and spec artifacts.

In Wix Stores, product categories (also labeled "Collections" in the UI) are a built-in first-class relationship: every product can belong to one or more categories, and every active store can have zero or more categories. The Category→Product shape is native and has table-level support in the Stores data schema. In Wix Bookings, service categories are a built-in grouping mechanism — providers assign services to categories in their catalog configuration. The Category→Services shape is native. In Wix Events, categories are less formally structured; events can be organized by label, tag, or recurrence — making Events a softer test case for RiR hierarchy.

## Competitive Landscape

- **Webflow:** Depth-3 with drag-and-drop, hard cap of **100 nested items per parent** and 10 nested Collection Lists per page. An ecosystem of third-party workarounds (Finsweet, jQuery patterns) exists to push past the hard caps — signaling even these limits are hit regularly by power users.
- **Bubble:** Unbounded depth, no item cap. Performance degrades exponentially ("each nested search multiplies database calls"); workload-unit pricing makes deep nesting expensive. Self-Creators struggle with the mental model.
- **Framer:** ~2 levels via slug-based filtering on dynamic pages. Not true inline nesting; no native "list inside a list on one page."
- **Squarespace:** No nested support. Users who need parent→child hierarchies migrate to Wix or Webflow.
- **Implication:** The spec's depth=3, max=100 mirrors Webflow's benchmark exactly. The question this investigation answers is whether the Wix-specific default (10) and warning threshold (200) are calibrated to the Wix user base — which may behave differently from Webflow's audience.

---

## Key Decisions to Make

*Calibration decisions (build is committed):*
- Is default=10 the right default for the nested item limit? — **must-resolve** (FR-016; directly visible UX impact on first bind)
- Is 200 the right multiplicative warning threshold for FR-017? — **must-resolve** (OQ-1; engineering is currently using 200 as a placeholder)
- Is depth=3 validated by real 3-level data hierarchies in Wix? — **should-resolve** (if rare, depth-2 v1 scope is lower risk)
- Which vertical should anchor the first RiR launch template / example? — **should-resolve** (Stores is the spec's canonical example; data should confirm the largest addressable base)
- How many active sites would immediately benefit from RiR? — **nice-to-know** (OQ-3 audience sizing; frames the feature's TAM)

---

## Questions to Investigate

### Opportunity sizing

---

**Q1. How large is the addressable base of Wix Stores sites with a Category → Product structure, and what are the cardinalities?**

- **Decision served:** Which vertical to prioritize for v1 template; how large is the Stores-specific RiR audience (OQ-3)
- **Metrics:**
  - **Metric:** Share of active Stores sites with ≥1 non-empty product category
    - **Population:** Sites (msids) with a Wix Stores app instance that processed ≥1 order in the last 90 days (Active store definition)
    - **Cohort filter:** Sites where the store has ≥1 product category containing ≥1 product
    - **Timeframe:** Current snapshot (as of query date)
    - **Logic:** COUNT(DISTINCT msid meeting filter) / COUNT(DISTINCT msid all active stores) → % + absolute count
  - **Metric:** Distribution of product category count per active Stores site
    - **Population:** Active Stores sites (≥1 order in last 90 days, has Stores app instance)
    - **Cohort filter:** All active stores (no additional filter — 0 categories is a valid data point)
    - **Timeframe:** Current snapshot
    - **Logic:** P10/P25/P50/P75/P90/P95 of category count per msid; histogram with buckets: 0 categories, 1, 2–3, 4–10, 11–25, 26+
  - **Metric:** Distribution of product count per category across active Stores sites
    - **Population:** (msid, category) pairs from active Stores sites where the category contains ≥1 product
    - **Cohort filter:** Exclude empty categories (0 products)
    - **Timeframe:** Current snapshot
    - **Logic:** For each (msid, category) pair, count distinct products. Report P10/P25/P50/P75/P90/P95 across all pairs; histogram with buckets: 1–5, 6–10, 11–25, 26–50, 51–100, 101+
  - **Metric:** Per-site multiplicative estimate: category count × in-site P50 products per category
    - **Population:** Active Stores sites with ≥2 non-empty categories
    - **Cohort filter:** Exclude stores with 0 or 1 category (no nesting benefit)
    - **Timeframe:** Current snapshot
    - **Logic:** For each msid: compute category_count × median(products_per_category across this site's categories). Report P25/P50/P75/P90 of this per-site estimate across all qualifying stores. This is the natural full-render cost if a builder shows all categories with their median item count.
  - **Metric:** Share of active Stores sites with evidence of a 3-level product hierarchy
    - **Population:** Active Stores sites
    - **Cohort filter:** Stores with any sub-category structure or category-within-category grouping
    - **Timeframe:** Current snapshot
    - **Logic:** % of active stores where product taxonomy has depth ≥ 3 (e.g., Department → Category → Product); note if the data schema supports this or if it must be inferred from naming conventions
- **Breakdowns:** By store product volume tier (1–10 products, 11–50, 51–200, 201+); by site creator type (Self-Creator vs. Partner / Contributor); by geography (top 5 markets by active store count)

---

**Q2. What is the valid Repeater in Repeater use case in Wix Events, and what are the cardinalities?**

- **Decision served:** Whether Events is a strong secondary vertical for RiR templates and examples; if yes, what cardinalities to use as reference
- **Metrics:**
  - **Metric:** Share of active Events sites with ≥2 distinct event categories or label groupings
    - **Population:** Sites with a Wix Events app instance that had ≥1 paid ticket transaction in the last 90 days (Active Events site definition)
    - **Cohort filter:** Sites where events are tagged with at least 2 distinct category labels / event types / series names
    - **Timeframe:** Current snapshot
    - **Logic:** COUNT(DISTINCT msid with ≥2 distinct labels) / COUNT(DISTINCT msid all active Events sites)
  - **Metric:** Distribution of event category / label count per active Events site
    - **Population:** Active Events sites (≥1 ticket transaction in last 90 days)
    - **Timeframe:** Current snapshot
    - **Logic:** P25/P50/P75/P90 of distinct category/label count per msid; histogram (1, 2–3, 4–10, 11+)
  - **Metric:** Distribution of events per category per active Events site
    - **Population:** Active Events sites with ≥2 distinct categories
    - **Timeframe:** Current snapshot
    - **Logic:** For each (msid, category) pair, count events. P25/P50/P75/P90 across all pairs; histogram
  - **Metric:** Share of active Events sites using recurring series (proxy for Event → Sessions use case)
    - **Population:** Active Events sites
    - **Logic:** % with ≥1 recurring event series; report median sessions per series for the recurring cohort
- **Breakdowns:** By events-per-site volume (small: 1–5, medium: 6–20, large: 20+); compare label-grouping vs. recurring-series patterns to determine which hierarchy is more common

---

**Q3. What is the valid Repeater in Repeater use case in Wix Bookings, and what are the cardinalities?**

- **Decision served:** Whether Bookings is a viable v1 template vertical; cardinality calibration for the Bookings use case
- **Metrics:**
  - **Metric:** Share of active Bookings sites with ≥1 service category containing ≥2 services
    - **Population:** Sites with a Wix Bookings app instance that had ≥1 booking confirmed in the last 90 days (Active Bookings site definition)
    - **Cohort filter:** Sites where services are organized into at least 1 named service category with ≥2 services
    - **Timeframe:** Current snapshot
    - **Logic:** COUNT(DISTINCT msid meeting filter) / COUNT(DISTINCT msid all active Bookings sites)
  - **Metric:** Distribution of service category count per active Bookings site
    - **Population:** Active Bookings sites (≥1 confirmed booking in last 90 days)
    - **Timeframe:** Current snapshot
    - **Logic:** P25/P50/P75/P90 of service category count per msid; histogram (1, 2–3, 4–10, 11+)
  - **Metric:** Distribution of services per category per active Bookings site
    - **Population:** Active Bookings sites with ≥1 non-empty service category
    - **Timeframe:** Current snapshot
    - **Logic:** For each (msid, category) pair, count services. P25/P50/P75/P90 across all pairs; histogram
  - **Metric:** Staff → Services cardinality as an alternative hierarchy
    - **Population:** Active Bookings sites with ≥2 staff members
    - **Timeframe:** Current snapshot
    - **Logic:** Median number of assigned services per staff member (where staff member has ≥1 service); P50/P90. Compare to service-per-category cardinality from metric 3 to determine which hierarchy is more data-rich.
- **Breakdowns:** By service volume (small: 1–5 services, medium: 6–15, large: 16+); by site creator type

---

### Spec calibration

---

**Q4. Does the spec's default nested item limit of 10 cover the P50 use case — and at what percentile does truncation first appear?**

- **Decision served:** Whether default=10 is right for FR-016 (SUB-006b) — directly visible on first bind
- **Metrics:**
  - **Metric:** P50 products per category in active Stores sites
    - **Source:** Q1 metric 3 result — extract the P50 value
    - **Logic:** The spec's claim: "default = 10 is conservative." If P50 > 10, the default cuts off the median user at first bind. If P50 ≤ 10, the default covers the median.
  - **Metric:** % of active-store (msid, category) pairs where product count exceeds the limit thresholds
    - **Population:** All (msid, category) pairs from active Stores sites with ≥1 product
    - **Timeframe:** Current snapshot
    - **Logic:** % of pairs where product count > 5; > 10; > 25; > 50; > 100. Produces a "cut-off distribution" — what % of real categories would show truncation at each limit setting.
  - **Metric:** P50 services per category in active Bookings sites
    - **Source:** Q3 metric 3 result — extract the P50 value
    - **Logic:** Same question for Bookings: does default=10 cover the median Service-per-category count?
  - **Metric:** P50 events per category in active Events sites
    - **Source:** Q2 metric 3 result — P50 value
    - **Logic:** Same for Events
  - **Metric:** Cross-vertical P90 item-per-category comparison
    - **Logic:** Take P90 from Q1 metric 3 (Stores), Q2 metric 3 (Events), Q3 metric 3 (Bookings). Report side-by-side. The highest-cardinality vertical indicates where users are most likely to hit the default limit first and need to raise it.
- **Breakdowns:** By product volume tier (same as Q1); by store size (GPV tier if available)

---

**Q5. Is the multiplicative warning threshold of 200 combined items calibrated to a sensible percentile of real Stores usage?**

- **Decision served:** Whether to keep, lower, or raise the FR-017 warning threshold (OQ-1 resolution)
- **Metrics:**
  - **Metric:** Distribution of per-site "natural render cost" (category count × in-site P50 products per category)
    - **Source:** Q1 metric 4 — the per-site multiplicative estimate
    - **Logic:** Report P25/P50/P75/P90 of this distribution. The P90 value is the key number: "at P90 complexity, a builder who shows all categories with P50-item nesting would render X items total."
    - **Purpose:** Tells us what the 200 threshold means in percentile terms — e.g., "200 fires for the top 5% of stores" or "200 fires for the top 40% of stores"
  - **Metric:** % of active Stores sites whose "full-render" (all categories × all products, no limit) would exceed 200 items
    - **Population:** Active Stores sites with ≥1 non-empty category
    - **Timeframe:** Current snapshot
    - **Logic:** For each msid: compute SUM of product counts across all categories (= "if the builder shows all products in all categories with no limit"). % of sites where this sum > 200; also > 100; > 500; > 1,000
    - **Purpose:** Measures the worst-case warning rate — "if a builder uses default=10 and the warning computes outer × nested, what % of stores would see the amber indicator at first configuration?"
  - **Metric:** Calibration analysis — at what threshold does the warning fire for ≤10% of stores?
    - **Logic:** From the per-site multiplicative estimate distribution (Q1 metric 4), find the value T such that 10% of sites exceed T when using default settings (outer items from their real category count × default nested limit of 10). This is the "fires for the top decile" threshold. Compare to the spec's 200.
  - **Metric:** % of active Stores sites with ≥25 product categories (the spec's worked example: "25 outer × 10 = 250 items")
    - **Population:** Active Stores sites
    - **Timeframe:** Current snapshot
    - **Logic:** % with category_count ≥ 25; also ≥ 10; ≥ 20; ≥ 50. Shows whether "25 categories" is a P90 or P99 scenario — determines how representative the spec's own worked example is.
  - **Metric:** Modeled warning fire rate at the current threshold (200) using real store configurations
    - **Population:** Active Stores sites with ≥2 non-empty categories
    - **Logic:** For each msid: compute category_count × 10 (default nested limit). % of sites where this product > 200. This is the precise warning fire rate "if every builder uses the default and shows all their categories."
- **Breakdowns:** By store product volume tier; by GPV tier (low/medium/high-revenue stores if accessible); note that high-revenue stores may have more categories and thus be more sensitive to the threshold

---

## Open Gaps

- **Events hierarchy ambiguity:** The right RiR use case for Events is not fully confirmed. The two candidates — "Event Categories → Events in category" (listing page) and "Event → Sessions" (multi-session event) — have different data sources and different cardinality profiles. Q2 covers both, but the interpretation of results depends on confirming the intended use case with the Events product team.
- **Bookings Staff → Services vs. Category → Services:** Both are valid RiR shapes in Bookings. Q3 covers both, but the more common data shape should drive which one is featured in the first Bookings RiR template.
- **3-level hierarchy real-world evidence:** The spec supports depth=3 (Outer → Nested → Deeply Nested). Q1 metric 5 looks for this in Stores, but the data schema may not natively encode 3-level product hierarchies. If no evidence is found, depth-3 may need to be validated via a different signal (user surveys, partner interviews).
- **Analytical KB unavailable:** No KB-derived table names, KPI definitions, or metric conventions were available in this pass. The metrics in this brief are self-contained but have no KB-anchored references to specific Wix data tables. A data team pass using OpenMetadata or the Stores/Events/Bookings analytical KBs would significantly improve the query precision.
- **Existing workaround base (prerequisite):** Q1–Q3 in `data-analysis-plan-repeater-in-repeater.md` measure how many sites already attempt the nested shape via workarounds. That investigation is a prerequisite for segmenting "sites that will upgrade from workaround" vs. "new users discovering the feature" when interpreting the RiR adoption curve.

---

## KB-Anchored Context

- **Relevant existing KPIs:** Not available — KB MCP was unreachable. Analytical KB not queried.
- **Relevant data tables:** No KB-derived table names available. Likely relevant: Stores product catalog table (products ↔ categories), Bookings service configuration table (services ↔ categories), Events table with label/category assignments, `wt_metasites.base` (site-level vertical flags), `prod.verticals.apps_dim` (app instance per site)
- **Key segments from KB:** Not available from KB. Segments used in this brief are from the population spectrum in metric-patterns.md: Active business (≥1 order/booking/ticket sale in last 90 days) as the primary population for all three verticals. Self-Creator vs. Partner/Contributor as the secondary segment dimension.

---

## Calibration Gate & Dependencies

**Calibration gate:** If P50 products per category (Q1 metric 3) > 10, raise the default nested item limit to match the P50 or P75 value before spec finalizes. If the modeled warning fire rate at 200 (Q5 metric 5) > 30% of active stores using default settings, raise the threshold — a warning that fires for 30%+ of first-time configurers is noise. If < 5% of active stores have ≥3 natural levels of product hierarchy (Q1 metric 5), consider shipping depth-2 in v1 with a clear upgrade path to depth-3.

**Dependencies:**
- Q4 depends on Q1 (uses Q1's per-category cardinality data directly — run as a derived query from Q1 results)
- Q5 depends on Q1 (uses Q1's per-site multiplicative estimate — same data pass)
- Q2 and Q3 are independent and can run in parallel with Q1
- Q5 metric 3 (calibration analysis) depends on Q1 metric 4 (per-site multiplicative estimate)

**Suggested next step:** This brief is structured for `ck-data-query` to pull data automatically from the Stores, Events, and Bookings data schemas. Run Q1 first (Stores cardinality) — it feeds Q4 and Q5. Q2 and Q3 (Events/Bookings) can run in parallel.

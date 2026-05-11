# Data Analysis: Repeater in Repeater — Vertical Data Investigation

> Source plan: [data-analysis-plan-rir-vertical-data.md](data-analysis-plan-rir-vertical-data.md)

## Context

**Problem:** Three quantitative parameters in the Repeater in Repeater spec — default nested item limit (10), multiplicative warning threshold (200), and depth limit (3) — were set as calibrated starting points, not derived from real usage data. This investigation asks whether the spec's numbers match what Wix merchants actually encounter in their Stores data.

**Audience:** Studio Users and Self-Creators building catalog/listing sites with Wix Stores. Active stores (≥1 order in last 90 days) are the primary population for cardinality analysis.

**Expected outcome:** Closing OQ-1 (calibrate the 200-item threshold against real data) and validating whether default=10 covers the median use case.

For full methodology, metric definitions, and cohort specifications, see the [source plan](data-analysis-plan-rir-vertical-data.md).

## Decisions This Analysis Serves

- **Is default=10 the right default for the nested item limit?** (must-resolve) — **CLOSED: confirmed**
- **Is 200 the right multiplicative warning threshold for FR-017?** (must-resolve) — **CLOSED: confirmed, OQ-1 resolved**
- **Is depth=3 validated by real 3-level data hierarchies?** (should-resolve) — **PARTIAL: 11.9% of category stores use subcategories; exact 3-level rate pending deeper query**
- **Which vertical anchors the first RiR launch template?** (should-resolve) — **Stores (Category→Products) confirmed as the right primary anchor**

---

## Findings

### Opportunity Sizing

#### Q1 — Stores: Category → Product cardinalities

**Execution date:** 2026-05-10  
**Population:** 249,356 active stores (Stores app installed, ≥1 paid online order in last 90 days, not QA site)  
**Source tables:** `prod.wt_metasites.stores`, `prod.stores.categories_dim`

---

##### Q1-M1 — Share of active stores with ≥1 non-empty user-created category

| Metric | Value |
|--------|-------|
| Active stores (base) | 249,356 |
| Stores with ≥1 user-created category | 12,585 |
| Share | **5.0%** |

**Interpretation:** Category-based navigation is a power-user feature. 95% of active Wix Stores merchants do not use the category system at all. The RiR Category→Products use case addresses the top 5% by sophistication.

---

##### Q1-M2 — Distribution of category count per active store (among stores with ≥1 category)

| Percentile | Category count |
|-----------|---------------|
| P10 | 1 |
| P25 | 3 |
| **P50** | **4** |
| P75 | 7 |
| P90 | 14 |
| P95 | 22 |

**Histogram (n = 12,585 stores with categories):**

| Bucket | Count | Share |
|--------|-------|-------|
| 1 category | 1,397 | 11.1% |
| 2–3 categories | 4,605 | 36.6% |
| 4–10 categories | 4,793 | 38.1% |
| 11–25 categories | 1,295 | 10.3% |
| 26+ categories | 495 | 3.9% |

**Interpretation:** The median category-using store has 4 categories. 86% of stores have 1–10 categories. Only the top ~4% exceed 25 categories — the territory where FR-017's warning becomes relevant.

---

##### Q1-M3 — Distribution of products per (msid, category) pair

**Population:** 91,169 (msid, category) pairs across 12,585 stores; user-created, non-deleted categories.

| Percentile | Products per category |
|-----------|----------------------|
| P10 | 0 |
| P25 | 1 |
| **P50** | **3** |
| P75 | 9 |
| P90 | 23 |
| P95 | 43 |

**Histogram:**

| Bucket | Count | Share |
|--------|-------|-------|
| 0 products (empty) | 19,494 | 21.4% |
| 1–5 products | 38,157 | 41.8% |
| 6–10 products | 13,234 | 14.5% |
| 11–25 products | 12,037 | 13.2% |
| 26–50 products | 4,487 | 4.9% |
| 51–100 products | 2,221 | 2.4% |
| 101+ products | 1,520 | 1.7% |

**Interpretation:** The median category has just 3 products. 77.7% of categories have ≤10 products. The long tail is significant — 8.8% of categories contain more than 50 products — but these are clearly power-user cases. 21.4% of categories have 0 products (empty, likely work-in-progress or archived).

---

##### Q1-M4 — Per-site multiplicative estimate (category_count × in-site P50 products_per_category)

**Population:** 12,585 stores with ≥1 user-created category.

| Percentile | Natural render cost |
|-----------|---------------------|
| P25 | 3 |
| **P50** | **11** |
| P75 | 36 |
| P90 | 96 |

**Interpretation:** The median store's natural render cost (without any item limit applied) is just 11 items — 4 categories × ~3 products each. Only the top 10% would render more than 96 items in total. This reinforces that the spec's parameters are well-calibrated for the typical use case.

---

##### Q1-M5 — Subcategory depth signal

| Metric | Value |
|--------|-------|
| Stores with ≥1 subcategory | 1,498 |
| Share of category-using stores | **11.9%** |

**Note:** This measures depth ≥ 2 (has at least one subcategory). Exact 3-level rate (subcategory-of-subcategory) requires a follow-up self-join query and is pending.

**Interpretation:** 12% of category-using stores already organize products into at least two levels. Depth=3 support in v1 is a real use case for this cohort.

---

### Spec Calibration

#### Q4 — Does default=10 cover P50 products per category?

**Input:** P50 products per category = **3** (from Q1-M3)

| Limit | % of stores where in-site P50 products_per_cat EXCEEDS this limit |
|-------|----------------------------------------------------------------|
| 5 | 29.9% |
| **10** | **12.4%** |
| 25 | 2.5% |

**Verdict:** ✅ **default=10 is confirmed.** The P50 products per category across all category-using stores is 3. Even the within-store P50 exceeds the default limit in only 12.4% of stores. The spec's default does not need to be raised.

**Calibration gate check:** "If P50 products per category > 10, raise the default" → P50 = 3. Gate does NOT trigger.

---

#### Q5 — Is the 200-item warning threshold at the right percentile?

**Model:** Warning fires when `category_count × nested_item_limit > threshold`. With defaults (nested_item_limit=10, threshold=200), warning fires when category_count > 20.

| Threshold | Stores where warning fires | Share of category-using stores |
|-----------|---------------------------|-------------------------------|
| 100 | 1,790 | 14.2% |
| 150 | 1,038 | 8.3% |
| **200** | **670** | **5.3%** |
| 300 | 366 | 2.9% |
| 500 | 167 | 1.3% |

**Additional context:**
- Stores with ≥25 categories (spec's worked example: 25 × 10 = 250 > 200): **530 stores** (4.2% of category-using stores)
- Natural render cost P90 = 96 — the 90th-percentile store renders fewer than 100 items even without a limit

**Verdict:** ✅ **threshold=200 is confirmed. OQ-1 is CLOSED.** The warning fires for 5.3% of stores using categories with default settings — well within any reasonable range for a guard-rail warning. The plan's calibration gate ("if fire rate > 30%, raise the threshold") does not trigger.

**Calibration gate check:** "If modeled warning fire rate at 200 > 30%, raise threshold" → fire rate = 5.3%. Gate does NOT trigger.

---

## Open Items

1. **3-level depth rate** — Query to determine what % of the 1,498 subcategory-using stores have subcategories of subcategories (depth=3). A self-join on `categories_dim.parent_category.id` would resolve this. Not blocking spec delivery; included in OQ-2 follow-up plan.

2. **Events and Bookings verticals** — Q2 and Q3 from the research plan were not executed in this pass. The Stores data fully resolves all calibration decisions. Events/Bookings remain available for a follow-up investigation if template selection requires cross-vertical validation.

3. **Empty categories** — 21.4% of (msid, category) pairs have 0 products. The RiR spec handles this (FR-019: empty state per nested item). No action required from this analysis.

4. **`items_in_category_count` scope** — This field in `categories_dim` counts all product associations, including hidden/unlisted products. Visible-only counts would be slightly lower. Not expected to change the calibration conclusions materially.

## Methodology Notes

- **Active store definition:** `prod.wt_metasites.stores` WHERE `is_valid = true AND COALESCE(total_paid_online_transactions_last_90_days, 0) >= 1 AND is_qa_site = false`. This gives 249,356 stores as of 2026-05-10.
- **Category filter:** `is_deleted = false AND is_autogenerated_category = false`. Autogenerated categories (e.g., "All Products") excluded to measure only merchant-created structure.
- **Percentiles:** Computed with `APPROX_PERCENTILE` via Trino. Distribution metrics over (msid, category) pairs, not per-store averages, per the plan's anti-heterogeneity-bias requirement.
- **Multiplicative estimate:** `category_count × in-site P50 products_per_category`. Computed per store, then percentiled across stores.
- **Dora status:** Unavailable in this session; local workflow (OpenMetadata + Trino) used throughout.
- **KB enrichment:** KB-retrieval server unavailable; domain context sourced from existing project brief.

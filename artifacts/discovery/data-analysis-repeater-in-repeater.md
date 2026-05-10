# Data Query Status — Repeater in Repeater (Studio 2 CMS)

**Project:** Repeater in Repeater (Studio 2 CMS)
**Phase:** Discovery — Data Query
**Date:** 2026-05-06

## Status

**Not executed in this pass.** OpenMetadata's table-search tool returned 404 in the current environment (`openmetadata__search_tables` and `openmetadata__search` both unavailable). Trino, KB retrieval, and `openmetadata__list_all_product_domains` are reachable, but without table discovery the analysis-plan queries would be guess-driven. Deferring to a data-team-led pass.

## Domains located

From `openmetadata__list_all_product_domains` (159 total domains). Most relevant for this project:
- Studio Editor (need to fetch domain UUID via paged listing)
- Wix Data (CMS) (same)

## Recommended follow-up

The full set of queries lives in [data-analysis-plan-repeater-in-repeater.md](data-analysis-plan-repeater-in-repeater.md). Highest-priority three for sizing the bet:

1. **Q1 — Workaround attempts.** Count Studio 2 sites with: 2+ Repeaters bound to the same dataset on one page, OR Repeater bound via secondary dataset to a multi-reference field, OR Repeater inside a dynamic page filtered by a reference field. Trailing 90 days. Slice by Self-Creator / Studio User / Partner.
2. **Q2 — CC volume.** Sum highlight count and unique reporters for the three User Voice clusters (`b4d71357…` multi-reference-in-repeater, `c9c4afa8…` shared-dataset filter, subset of `a14ad01d…` multiple-repeaters spacing) per quarter, trailing 12 months.
3. **Q3 — Publish funnel.** Among first-time workaround sites, 60-day publish rate and 30-day return-to-edit rate vs. matched control of CMS-Repeater sites without the workaround.

## What this means for the PRD

Quantitative sizing is unblocked once the data team runs the plan. The qualitative case (UV research + Help Center request page + Studio Forum threads + Wix Blocks parity) is already strong enough to proceed with PRD drafting; the data pass will calibrate scope and prioritize verticals (Stores categories→products vs. Restaurants meals→dishes vs. Bookings services→staff, etc.).

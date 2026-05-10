# Project Brief: Repeater in Repeater — Vertical Data Investigation

> Pre-Research Stage — Everything here reflects initial assumptions and early analysis. Nothing has been validated yet.

## Context

Studio 2 CMS, Repeater in Repeater feature. This investigation validates whether the product definitions in [`artifacts/product/product-spec-repeater-in-repeater.md`](product/product-spec-repeater-in-repeater.md) match real-world usage patterns across Wix verticals.

Key spec parameters under investigation:

| Parameter | Spec value | Source |
|---|---|---|
| Default nested item limit | 10 per parent row | SUB-006b, FR-016 |
| Max nested item limit | 100 | SUB-006a |
| Multiplicative warning threshold | 200 (outer × nested) | FR-017, OQ-1 |
| Depth limit | 3 levels | Overview §1 |

OQ-1 in the spec reads: *"calibrate the threshold against real benchmark data."* This investigation directly addresses it. OQ-3 ("size the affected base") is a secondary benefit.

## Direction

1. **Identify valid Repeater in Repeater use cases** from three Wix verticals:
   - **Stores**: Category → Products (the canonical example in the spec)
   - **Events**: Identify the correct hierarchical relationship and cardinality (e.g., Event → Sessions / Ticket types)
   - **Bookings**: Identify the correct hierarchical relationship and cardinality (e.g., Service Category → Services)

2. **Query real cardinality data** for each valid use case:
   - Stores: P50 and P90 # of categories per store site; P50 and P90 # of products per category
   - Events / Bookings: equivalent metrics once the use cases are confirmed

3. **Cross-reference against spec defaults** to validate or challenge:
   - Is default = 10 correct? Does the median category have ≤10 products?
   - Is the 200-item warning threshold sensible? Does the P90 outer × nested product count exceed 200?
   - Is depth = 3 sufficient? Are there real 3-level hierarchies in Wix user data?

## Goal

Validate an idea — confirm whether the spec's item limit default (10), warning threshold (200), and depth limit (3) are calibrated to real user data rather than estimated heuristics. Secondarily, size the addressable audience for the feature (OQ-3).

## Target Group

Internal PM data investigation. End-user context (once validated): Studio Users and Self-Creators building catalog, listing, or booking sites using Wix Stores, Events, or Bookings.

## More

- The spec's worked example for OQ-1: "25 outer × 10 nested = 250 items" needs grounding in data. Is 25 categories a P90 or a P99 value? Is 10 products per category the median or a generous ceiling?
- Comparison anchor: Webflow hard-caps nested lists at 5 with no builder control. Wix's default of 10 is already more permissive. The question is whether 10 covers the median use case without forcing the user to change it immediately.
- Depth-3 validation: the spec supports Outer → Nested → Deeply Nested (3 levels). Are there real Wix data structures that are naturally 3 levels deep? (e.g., Department → Category → Product in a large store?)
- Sites-affected count: include an estimate of how many active Wix sites already have the data shape that would benefit from Repeater in Repeater, to help size OQ-3.

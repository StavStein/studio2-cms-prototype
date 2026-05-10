# Project Brief: Repeater in Repeater (Studio 2 CMS)

> Pre-Research Stage — Everything here reflects initial assumptions and early analysis. Nothing has been validated yet.

## Context

Studio 2 editor, CMS data binding surface. Builders today can connect a top-level Repeater to a context and array, design item layouts, and bind inner elements. The next step in the data-binding story is the **nested Repeater** — a Repeater placed inside another Repeater's item, where the inner Repeater is driven by a list field of the parent item (e.g. an `episodes` array on each `season`, or a `lessons` array on each `course`).

The prototype already includes nested-repeater plumbing (`renderSettingsRepeaterOnRepeater()` is a distinct inspector render path), and the existing blank-Repeater PRD acknowledges nested arrays as a selectable type but does not specify the nested experience end-to-end. There is no dedicated PRD for this case yet.

## Direction

Produce a standalone PRD for nested Repeater behavior, at the same level of depth as `blank-repeater-prd.md`. Cover the full lifecycle: initial state, context inheritance from the parent item, attaching a context vs. binding `Items` from the parent item's array fields, the binding picker (scope badges, parent-item context affordances), inner bindings inside the inner item, replace/disconnect flows, and edge cases (mismatched sources, no nested array, deep nesting limits, empty inner arrays). Highlight where nested differs from top-level and where it shares behavior.

## Goal

- **Solve a user pain point** — nested data scenarios (categories with products, courses with lessons, seasons with episodes) are clumsy or impossible without nested Repeaters and deserve a clean, predictable model.
- **Close a competitive gap** — match the nested-list patterns that Webflow, Framer, Bubble, and Squarespace builders expect.
- **Formalize existing prototype** — document the established direction in the binding-platform demo so engineering and design have a shared spec to build against.

## Target Group

- **Studio User** (primary) — agencies and advanced creators using Studio 2 to build data-driven sites for clients.
- **Self-Creator** — Wix users on Studio 2 who manage their own data-rich content (catalogs, programs, listings).
- **Partner (agency / freelancer)** — building structured-content sites for clients where nested collections are common.

## More

- The nested-Repeater PRD should consume and reference the existing artifacts rather than restating them: `blank-repeater-prd.md` (connection model, context attach, replace/disconnect patterns), `artifacts/product/product-spec-repeater-pagination.md` (pagination behavior already differentiates nested vs. standard via `renderSettingsRepeaterOnRepeater`), and the discovery set under `artifacts/discovery/` for repeater event-handler context.
- Open questions to call out explicitly in the PRD: depth limit (2 levels? unbounded?); whether the inner Repeater can attach its own context independent of the parent item's array; how the binding picker labels parent-item array fields vs. page-level contexts (`This item` vs. `This repeater` vs. page scope); and how disconnect cascades when the parent Repeater's `Items` is changed or removed.
- Worth confirming before drafting: scope of "AI interactions" inclusion (the blank-Repeater PRD defers AI to a separate section — apply the same separation here).

## Domain References

| Domain | KB ID |
|--------|-------|
| Studio Editor | b4105bf2-66c7-4a75-8934-5c4ab23ebc19 |
| Wix Data (CMS) | 781cbc83-58f3-4ea3-b436-fe81b2f799d6 |

## Domain Gameplan

https://docs.google.com/document/d/1ZtX6ZE1AbJXCPh5Frfsx5LwnB88Y_DTB1J4d1ViiN7o/edit?tab=t.0#heading=h.9blq0q1gist

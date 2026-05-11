# Terminology Guide — Repeater in Repeater (Studio 2 CMS)

**Date:** 2026-05-06
**Domain:** Studio 2 Editor / Wix Data (CMS) / Binding Platform
**Based on:** Competitor research (Webflow, Framer, Bubble, Squarespace, Wix Blocks), User Voice research (support tickets, Wix Studio Forum, Velo Forum), Internal discovery (existing PRDs and prototype), the existing repeater-event-handlers terminology guide. Wix UX Writing Glossary API not queried in this pass — flagged as a manual check.

---

## How to Use This Guide

This is the source of truth for naming on the nested-Repeater PRD. Use these terms in the PRD, in UX concepts, in the Inspector, in the floating binding panel, in canvas copy, and in dev docs. When in doubt, use the **Recommended term**.

---

## Agreed Terminology

### Objects

| # | Concept | Recommended term | Wix terminology baseline | Definition | Competitor terms | User language | Notes |
|---|---|---|---|---|---|---|---|
| 1 | A Repeater placed inside another Repeater's item template | **Nested Repeater** | New — extends "Repeater" | A Repeater that lives inside another Repeater's item template and binds its `Items` to the parent item's array or multi-reference field | Webflow: "Nested Collection List"; Bubble: "Nested Repeating Group"; Framer: "nested CMS list" | "repeater inside repeater," "repeater in repeater," "child repeater," "sub-repeater" | **Use "Nested Repeater" in PRD and inspector copy.** Avoid "child Repeater" — overloaded with parent/child item semantics |
| 2 | The Repeater that contains the nested Repeater | **Outer Repeater** | New | The enclosing Repeater whose item template contains the Nested Repeater | Webflow: "parent Collection List"; Bubble: "parent Repeating Group" | "outer one," "main repeater," "parent repeater" | "Outer" reads cleaner than "parent" because the items themselves are also called parent rows — minimizes ambiguity |
| 3 | One row of the Outer Repeater | **Parent item** | Extends Wix "Repeater item" | A single rendered row of the Outer Repeater. Holds the data context that drives its Nested Repeater | Webflow: "parent collection item"; Bubble: "parent cell" | "the row," "the parent," "the meal" / "the category" / domain-specific | Use in inspector and panel copy. "Parent item" pairs naturally with "child items" |
| 4 | A row of the Nested Repeater | **Child item** | Extends Wix "Repeater item" | A rendered row inside a Nested Repeater. Bound to one entry of the Parent item's array / multi-reference field | Webflow: "nested collection item" | "sub-item," "inner row," "the dish" / "the lesson" / domain-specific | "Child" is fine here because the relationship is data-shaped (parent collection → child collection) |
| 5 | Array or multi-reference field on the parent item that drives the Nested Repeater | **Child array** (general) / **Child collection** (when the field is a multi-reference) | Extends Wix "multi-reference field" / "array field" | The list-shaped field on the parent collection that the Nested Repeater binds to | Webflow: "linked collection field"; Bubble: "list field" | "list field," "the related items," "the multi-reference," "the sub-collection" | Use **Child array** in generic copy and Velo docs, **Child collection** when specifically referencing a multi-reference relationship in CMS |
| 6 | The data row that the Nested Repeater inherits from its Parent item | **Parent row context** (or short: **Parent context**) | New scope concept; sibling to existing "page-level context" and "this Repeater context" | The single data row of the parent collection that scopes everything inside the parent item template — including the Nested Repeater's `Items` source | Webflow: "current item"; Bubble: "current cell"; Framer: "current item" | "current row," "this row," "the parent's data" | Internal/PRD term. The user-facing label in the binding picker is the badge **`This item`** (see Features § 1) |

### Actions

| # | Concept | Recommended term | Definition | Competitor terms | User language | Notes |
|---|---|---|---|---|---|---|
| 1 | Connecting the Nested Repeater's `Items` to a Child array of the Parent item | **Bind to parent item** (verb) / **Connect to parent item** (UI) | Wires the Nested Repeater's `Items` to a list-shaped field of the parent collection, scoped via the Parent row context | Webflow: "nest a Collection List"; Bubble: "set data source to current cell's list" | "connect inner repeater," "wire to parent" | The PRD uses "bind to parent item." UI copy uses "Connect to parent item" — matches the existing `Connect data` button family |
| 2 | Inheriting the Parent context automatically when an inner Repeater is added | **Inherit parent context** | When a Repeater is dropped inside another Repeater's item, it gains access to the Parent row context without an explicit attach action | Webflow: implicit; Bubble: explicit; Framer: implicit | "auto-pick up the row," "knows about the parent" | Used in PRD only — not user-facing copy |
| 3 | Removing the Nested Repeater's binding | **Disconnect Nested Repeater** | Explicit user action to unbind the Nested Repeater's `Items` while preserving its layout and item-level styling | Webflow: "remove binding"; Bubble: "clear data source" | "unbind," "remove connection," "disconnect" | Confirmation copy must call out grandchild-binding cascade |
| 4 | Switching the Parent item's `Items` while a Nested Repeater is connected | **Replace Outer Repeater context** | User changes the Outer Repeater's `Items` source; cascades down to the Nested Repeater | Webflow: "switch source"; Bubble: "change data source" | "swap collection," "change source" | Carries the existing replace-context warning pattern from the blank-Repeater PRD; the warning must include nested-binding count |

### Features & Capabilities

| # | Concept | Recommended term | Definition | Competitor terms | User language | Notes |
|---|---|---|---|---|---|---|
| 1 | The binding-picker scope label for fields surfaced from the Parent row context | **`This item`** | A scope badge in the binding panel that marks fields drawn from the current parent item, distinct from `This repeater` and page-level scopes | Webflow: implicit (no badge — fields just appear); Bubble: "Current cell's …"; Framer: "Current Item" | "this row's stuff," "the parent's fields" | **Reuse the badge pattern** from the existing `This repeater` work. `This item` ranks above `This repeater` and page-level when the picker is opened from a Nested Repeater |
| 2 | The maximum nesting depth supported by the canvas binding flow | **Nesting depth** (concept) / **Depth limit** (UI label when surfacing the cap) | The number of levels of Repeater nesting the canvas binding flow supports. v1 recommendation: 2 | Webflow: 3 levels; Bubble: unbounded | "how deep," "nesting" | Surface the limit in the inspector when reached. Don't hard-fail silently |
| 3 | Dedicated inspector render path for nested Repeaters | **Nested Repeater inspector** | The branch of the inspector that renders when the selected Repeater is itself inside another Repeater's item | n/a | n/a | Existing prototype function: `renderSettingsRepeaterOnRepeater()`. Internal/PRD-only term — the user does not see this label |
| 4 | The mismatch between attached context and `Items` source | **Context mismatch** | Existing concept from blank-Repeater PRD §10. Extends to nested: when a Nested Repeater has an attached context that differs from its Parent row context | n/a | "different sources" | Reuse the blank-Repeater PRD §10 framing; add nested-specific examples |
| 5 | Cap on items rendered per parent item | **Items-per-parent cap** | Soft cap on how many child items a Nested Repeater renders per parent item (Webflow uses 100). Surfaced in inspector with a clear message | Webflow: "100 items per parent" | n/a | Engineering-bounded; surface in inspector |

### Data Shape Terms

| # | Concept | Recommended term | Definition | Competitor terms | User language | Notes |
|---|---|---|---|---|---|---|
| 1 | A field on a CMS collection that points to a row in another collection | **Reference field** | Wix CMS canonical | Webflow: "Reference field"; Bubble: "linked data" | "reference" | Established Wix term; keep |
| 2 | A field on a CMS collection that points to many rows in another collection (many-to-many) | **Multi-reference field** | Wix CMS canonical | Webflow: "Multi-reference field" | "multi-ref," "many-to-many" | Established Wix term; **the dominant data shape behind Nested Repeaters** |
| 3 | The list of rows resolved from a multi-reference field at runtime | **Resolved children** (PRD-internal) | The set of rows the runtime returns when resolving a multi-reference field for a single parent row | Webflow: "linked items"; Bubble: "list of [type]" | "the children," "the linked records" | PRD-only; UI uses concrete domain wording (dishes, products, lessons) |
| 4 | The runtime primitive for fetching referenced rows efficiently | **`include()`** (Blocks) / **`queryReferenced()`** (Velo) | Existing Velo / Blocks API that batches the cross-collection fetch | n/a | n/a | Engineering-facing only; **the primitive Studio 2 canvas runtime should reuse to avoid N+1 queries** |

---

## Terms to Avoid

| Don't use | Use instead | Why |
|---|---|---|
| "Sub-Repeater" | **Nested Repeater** | "Sub" is ambiguous in this domain (sub-collection? sub-item? sub-section?); "Nested" is unambiguous |
| "Inner Repeater" / "Child Repeater" | **Nested Repeater** | "Child" is overloaded — child item, child row, child collection — and "inner" is too informal |
| "Collection List" | **Repeater** (or **Nested Repeater**) | Webflow term; importing it creates parity confusion |
| "Repeating Group" | **Repeater** | Bubble term |
| "Sub-collection" (in UI copy) | **Child collection** (when discussing the multi-reference data shape) | "Sub-collection" implies a hierarchy in the data model that doesn't exist in CMS |
| "Repeater within Repeater" / "Repeater inside Repeater" | **Nested Repeater** | These are user-language phrasings — fine in Help Center search hints, not in product copy |
| "Linked Repeater" | **Nested Repeater** | "Linked" implies a different mechanism (URL/page link); "Nested" is the structural relationship |
| "Group" (for the Outer Repeater) | **Outer Repeater** | "Group" already names the layout container in Studio Editor |

---

## Open Questions

| # | Term / Concept | Question | Options |
|---|---|---|---|
| 1 | Depth limit ceiling for v1 | Cap at 2 or 3 levels? | **2 (recommended)** for shipping speed and runtime guarantees; **3** to match Webflow exactly |
| 2 | Naming for the v1 binding-picker scope of the parent row | Confirm `This item` vs. alternatives | **`This item` (recommended)** — pairs with existing `This repeater`. Alternatives: `Parent item`, `Current item` |
| 3 | Whether "Outer Repeater" / "Nested Repeater" appears in user-facing UI | Should the inspector label the relationship explicitly? | **Yes for the Nested Repeater header / hat** — disambiguates state. **No for the Outer Repeater** — it's just a Repeater unless the context demands the distinction |
| 4 | Items-per-parent cap value | Match Webflow's 100, or set our own? | Pending engineering input; recommend 100 to start as a parity move |
| 5 | UI copy for the parent's child-array field type indication | Should fields on the parent item be marked as "list" / "many" in the picker? | **Yes** — parity with the existing array-field label hierarchy from blank-Repeater PRD §5 |

---

## Glossary Cross-Reference Summary

| Term | Status | Domain | Action |
|---|---|---|---|
| Repeater | ✅ Canonical (Velo) | Velo | Keep |
| Repeater item | ✅ Canonical | Velo | Use as base for Parent item / Child item |
| Multi-reference field | ✅ Canonical | Wix CMS | Keep |
| Reference field | ✅ Canonical | Wix CMS | Keep |
| Context | ✅ Already used in binding platform | Studio 2 binding | Extend with **Parent row context** |
| `This repeater` (scope badge) | ✅ Already specified in blank-Repeater PRD | Studio 2 binding | Sibling to new `This item` badge |
| `Items` (Repeater property) | ✅ Already specified in blank-Repeater PRD | Studio 2 binding | Reused for Nested Repeater binding |
| Nested Repeater | 🆕 New | Studio 2 | Add to glossary as a new term |
| Outer Repeater | 🆕 New | Studio 2 | Add as relational term |
| Parent item / Child item | 🆕 New | Studio 2 | Add as relational pair |
| Parent row context | 🆕 New (PRD-internal) | Studio 2 binding | Add as scope concept; user-facing label is `This item` |
| `This item` (scope badge) | 🆕 New | Studio 2 binding | Add as third scope sibling to `This repeater` and page-level |
| `include()` / `queryReferenced()` | ✅ Canonical (Blocks / Velo) | Wix Blocks / Velo | Engineering-facing; PRD references these as the runtime primitive |

---

## Recommended Manual Check

- **Wix UX Writing Glossary API** (VPN required) — confirm the new terms (`Nested Repeater`, `Outer Repeater`, `Parent item`, `Child item`, `This item`) don't collide with prohibited terms in the Studio Editor or Wix CMS domain. The repeater-event-handlers terminology guide flagged "interaction" as prohibited in Studio Editor; verify nothing similar applies here.

---

## Sources

- Competitor research: [`artifacts/discovery/competitor-research-repeater-in-repeater.md`](competitor-research-repeater-in-repeater.md)
- User Voice research: [`artifacts/discovery/user-voice-repeater-in-repeater.md`](user-voice-repeater-in-repeater.md)
- Internal discovery: [`artifacts/discovery/internal-discovery-repeater-in-repeater.md`](internal-discovery-repeater-in-repeater.md)
- Project brief: [`artifacts/project-brief-repeater-in-repeater.md`](../project-brief-repeater-in-repeater.md)
- Blank Repeater PRD: [`blank-repeater-prd.md`](../../blank-repeater-prd.md)
- Pagination spec: [`artifacts/product/product-spec-repeater-pagination.md`](../product/product-spec-repeater-pagination.md)
- Existing repeater-event-handlers terminology guide: [`artifacts/discovery/terminology-research-repeater-event-handlers.md`](terminology-research-repeater-event-handlers.md)

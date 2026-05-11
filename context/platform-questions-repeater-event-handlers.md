# Repeater Event Handlers — Open Questions for Platform & Velo Teams

We're defining the Repeater event model for Studio 2. Before we can finalize the spec, we need answers to the following. Please respond inline or schedule a sync.

---

## 1. Item scoping: does the right item's data reach the handler?

**User intent:** A developer clicks a button inside Repeater item #3 and expects the bound function to receive item #3's data — not item #1's.

**Gap:** After the legacy→dynamic event handler migration in Sept 2024, `$item` inside Repeater event handlers always returns the first item's data, regardless of which item was clicked. The workaround — `$w.at(event.context)` — exists but is non-obvious and not prominently documented.

**Question:** Is this regression fixed, still live, or intentional? If still live, is `$w.at(event.context)` the official recommended pattern going forward?

---

## 2. Reacting when item data changes

**User intent:** A developer updates a Repeater item's data at runtime (e.g., a live score changes) and wants a function to run automatically for that specific item — without writing manual boilerplate.

**Gap:** There is no `onItemUpdated` event. Setting `.data` on an existing item does not re-trigger `onItemReady` for that item. Every developer who needs to react to data changes has to manually call `forEachItem()` or `forItems()` after every update.

**Question:** Is `onItemUpdated` (or an equivalent) on the Velo roadmap? If so, what's the expected timeline, and who owns the API contract?

---

## 3. Hover and mouse events on dataset-connected Repeaters

**User intent:** A partner builds a portfolio grid where each card shows an overlay on hover. The Repeater is connected to a CMS dataset — the standard production setup.

**Gap:** `onMouseIn` / `onMouseOut` on child elements inside a Repeater stop firing silently the moment the Repeater is connected to a dataset. No error is thrown. Partners either drop the dataset connection or drop the hover effect — both are unacceptable trade-offs.

**Question:** What is the root cause — is it the dataset layer suppressing events, or the Repeater event system failing to wire them when a dataset is connected? Is this a tracked bug? Is a fix in progress?

---

## 4. Where do Repeater event properties live in the Studio 2 Inspector?

**User intent:** A developer selects a Repeater in the Studio 2 Inspector and expects to see its event handlers right there — without switching to a separate Code panel.

**Gap:** Today, `onItemReady` and `onItemRemoved` are only accessible from the Properties & Events panel in the Code workspace — not from the Inspector. The feature request to move them to the Inspector is open and in "Collecting votes" status.

**Question:** Who owns the Inspector event panel surface in Studio 2 — the CMS group or editor infrastructure? Are there architectural constraints on what we can expose there?

---

## 5. Alignment with the Wix Blocks Dynamic Repeater

**User intent:** A developer using both Studio 2 and Wix Blocks expects consistent Repeater behavior across both surfaces.

**Gap:** Wix Blocks has a Dynamic Repeater variant with a potentially different event contract and architecture. If Studio 2 defines its Repeater event model independently, we risk fragmentation across Wix's Repeater variants.

**Question:** Does the Blocks Dynamic Repeater use the same `onItemReady` / `onItemRemoved` model? Has it resolved any of the scoping issues we're seeing? Should Studio 2 align with it, or are the two intentionally divergent?

---

*Owner: Studio 2 / CMS group*
*Related spec: `artifacts/product/product-spec-repeater-event-handlers.md`*

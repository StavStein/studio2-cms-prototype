# Terminology Guide: Repeater Event Handlers — Studio 2 Editor

**Date**: 2026-05-05
**Domain**: Studio 2 Editor / Velo / CMS
**Based on**: Competitor research (Webflow, Framer, Bubble), User Voice research (Wix Community Forum, dev.wix.com), Internal Discovery (Velo API docs, Studio Editor docs), Wix UX Writing Glossary (retrieved via API — VPN required)

---

## How to Use This Guide

This is the source of truth for naming in this project. Use these terms in:
- PRDs and product specs
- UX concepts and wireframes
- Final UI copy and design
- Developer documentation and code

When in doubt, use the **Recommended term** column.

---

## Agreed Terminology

### Objects (things users create or manage)

| # | Concept | Recommended term | Wix Glossary | Definition | Competitor terms | User language | Notes |
|---|---------|-----------------|-------------|------------|-----------------|---------------|-------|
| 1 | The dynamic list component | **Repeater** | ✅ Canonical (Velo domain): "Editor elements that let you define one layout for a group of items and reuse that layout for each item while displaying different content." Use lowercase in UX copy; capitalize when referring to the official feature. | A layout element that displays a list of items using a shared template design, where each item can have different content | Webflow: "Collection List"; Bubble: "Repeating Group"; Framer: "CMS Collection List" | "repeater", "list", "dynamic list" | Wix established term; do not change to match competitors |
| 2 | A single entry in the Repeater | **Repeater item** | ✅ "item" is canonical in Velo/eComm contexts. Note: "item" is a **prohibited variant** for grid cells (use "Cell" for grid children). For Repeater entries, "item" is correct. | One instance of the Repeater template with its own content | Webflow: "Collection item"; Bubble: "Repeating Group cell"; Framer: "collection list item" | "item", "row", "card" | Use "Repeater item" in docs; "item" is acceptable shorthand in UI context. Distinguish from grid cells where "Cell" is the canonical term. |
| 3 | The design template shared by all items | **Item template** | — (no direct glossary match for Repeater-specific template) | The layout and design definition that applies to all items in a Repeater | Webflow: "Collection list item (template)"; Bubble: "Repeating Group cell template" | "template", "item layout" | Used in Wix docs; keep as is |
| 4 | Repeater connected to a CMS collection | **Dataset-connected Repeater** | — (no direct glossary match) | A Repeater whose item content is driven by a Wix CMS collection via a dataset | Webflow: "CMS-bound Collection List"; Framer: "CMS Collection List" | "connected repeater", "database repeater", "dynamic repeater" | Use "dataset-connected" for precision; "dynamic Repeater" is acceptable in conversational/UI context |

### Actions (what users do)

| # | Concept | Recommended term | Wix Glossary | Definition | Competitor terms | User language | Notes |
|---|---------|-----------------|-------------|------------|-----------------|---------------|-------|
| 1 | Connecting an event callback to a Repeater | **Add event handler** | ✅ "Event handler" is canonical (Velo domain): "A function that contains the code you want to run when a specific event occurs on an element." | Attaching a function to a Repeater lifecycle event or item event | Webflow: "Add interaction / trigger"; Bubble: "Add workflow action"; Framer: "Add event" | "wire up event", "add event", "connect handler" | Consistent with Wix Properties & Events panel language |
| 2 | Updating items without recreating them | **Update item data** | — | Setting new content for an existing Repeater item without triggering item creation lifecycle | Webflow: no equivalent (interactions are display-only); Framer: "update variant state" | "update repeater", "refresh item", "set data" | Use in context of the missing `onItemUpdated` spec decision |
| 3 | Iterating over all Repeater items in code | **Iterate items** | — | Programmatically looping through all or specific items to apply updates | Webflow: no equivalent; Framer: no equivalent | "loop through items", "for each item" | `forEachItem()` / `forItems()` are the API method names; "iterate items" is the plain-language equivalent for docs |

### Features & Capabilities

| # | Concept | Recommended term | Wix Glossary | Definition | Competitor terms | User language | Notes |
|---|---------|-----------------|-------------|------------|-----------------|---------------|-------|
| 1 | Events fired by the Repeater based on data state changes | **Repeater lifecycle events** | ✅ "Event" is canonical (Velo): "An action or change that happens to an element, usually triggered by a visitor, that your code can respond to." The "lifecycle" qualifier is our addition to distinguish from user-triggered events. | Callbacks that fire when Repeater items are created, updated, or removed in response to data changes — not user action | Webflow: no equivalent; Bubble: no equivalent; Framer: no equivalent | "lifecycle hooks", "data events", "item callbacks" | This is a Wix-unique capability; "lifecycle" is the right framing to distinguish from user-triggered events |
| 2 | Event fired when a new Repeater item is rendered | **onItemReady** | — (API method name, not a UI/copy term) | Callback that fires when a new item is created in the Repeater (triggered by data assignment or dataset connection) | No direct equivalent in Webflow/Bubble/Framer | "item ready event", "item created callback" | Established Velo API term; use as-is in developer-facing copy |
| 3 | Event fired when a Repeater item is removed | **onItemRemoved** | — (API method name, not a UI/copy term) | Callback that fires when an item is removed from the Repeater (triggered by data update); does NOT fire for statically-defined editor items | No direct equivalent in competitors | "item removed event", "item deleted callback" | Established Velo API term; panel copy should clarify the static-item exception |
| 4 | Proposed event for item data changes | **onItemUpdated** | — (does not exist yet) | (Proposed) Callback that would fire when an existing Repeater item's data is updated in place — does not exist today | No competitor equivalent | "item update event", "item changed callback", "data changed" | **Open question** — see Open Questions section. API doesn't exist yet; this is the recommended name if built. Alternative: `onItemDataChanged` |
| 5 | Events triggered by user action on items | **Item events** | ⚠️ CONFLICT: "Interaction" is canonical in Velo ("Custom functionality that lets your site respond to events, such as clicks, hover states") but is **prohibited** in Studio Editor domain ("Use 'Animations & Effects' instead of 'Interactions' when referring to these visual effects"). Since Studio 2 spans both, "Item interaction events" risks conflating event handlers with Animations & Effects. | User-initiated events (click, hover, mouse, keyboard) that fire on elements inside a Repeater item | Webflow: "Interactions / triggers"; Framer: "variant states / events"; Bubble: "workflow actions" | "click event", "hover event", "interaction" | **Changed from "Item interaction events"** — "interaction" is reserved for visual animations in Studio Editor context. Use "Item events" in UI copy; use "event handler" in developer-facing copy. Avoid "interaction" to prevent confusion with Animations & Effects. |
| 6 | The visual panel for connecting event handlers in Studio editor | **Properties & Events panel** | — (UI panel name, not in glossary) | The editor panel where developers can visually add event handlers to elements without writing code directly | Webflow: "Interactions panel"; Framer: "Properties panel"; Bubble: "Workflow editor" | "events panel", "code panel", "properties panel" | Established Wix term; may need to be integrated into the Studio 2 Inspector rather than a separate panel — naming TBD pending architecture decision |
| 7 | The selector scoped to a single Repeater item in code | **Item selector ($item)** | — (code/API term, not in glossary) | A `$w`-scoped selector that targets elements within a specific Repeater item during event handling | No direct equivalent in visual editors; React: closure-scoped reference | "$item", "scoped selector", "item context" | Developer-facing term; `$item` is the code name; "item selector" is the plain-language equivalent for docs |

---

## Terms to Avoid

| Don't use | Use instead | Why |
|-----------|------------|-----|
| "Collection List" | **Repeater** | Webflow terminology; using it in Wix context creates confusion |
| "Repeating Group" | **Repeater** | Bubble terminology; not Wix |
| "Dynamic list" | **Repeater** (or "dataset-connected Repeater" if CMS-specific) | Too generic; loses the Wix component identity |
| "Lifecycle hook" | **Lifecycle event** | "Hook" is React/framework jargon; "event" is more accessible and consistent with Wix documentation style |
| "Item callback" | **Event handler** (or the specific event name, e.g. **onItemReady**) | "Callback" is internal/developer jargon; "event handler" is more accessible and consistent with the Properties & Events panel language |
| "Repeater cell" | **Repeater item** | "Cell" implies a table/grid (Bubble usage); "item" is the established Wix term. Wix Glossary confirms "Cell" is the prohibited variant for grid children — use "Cell" for grid, "item" for Repeater. |
| "Data event" | **Lifecycle event** | Too vague; "lifecycle" communicates that the event is tied to the item's state (created/updated/removed) |
| "Interactions" (in Studio Editor UI context) | **Item events** (for user-triggered) or **Lifecycle events** (for data-triggered) | ⚠️ **Glossary finding**: In Studio Editor domain, "Interactions" is **prohibited** — it's reserved for Animations & Effects. Do not use "interactions" in Studio 2 UI copy to mean event handlers. Developer-facing Velo documentation may use "interaction" per Velo glossary. |
| "Item interaction events" | **Item events** | Renamed to avoid "interaction" which is prohibited in Studio Editor context per Wix UX Writing Glossary |

---

## Open Questions

| # | Term / Concept | Question | Options being considered |
|---|---------------|----------|------------------------|
| 1 | The new update lifecycle event | What should the new event be named? | **`onItemUpdated`** (mirrors `onItemReady` / `onItemRemoved` naming pattern) vs. **`onItemDataChanged`** (more explicit about what triggers it) |
| 2 | Properties & Events panel location in Studio 2 | Should it remain a separate panel or be integrated into the Inspector? | If integrated: "Events" section in Inspector; if separate: keep "Properties & Events panel". Naming depends on architecture decision. |
| 3 | "Dataset-connected Repeater" vs. "dynamic Repeater" | Should the CMS-connected variant be named differently from a static Repeater? | "Dataset-connected Repeater" (precise, matches existing Wix terminology) vs. "Dynamic Repeater" (simpler, matches competitor language) — the Wix Blocks framework already uses "Dynamic Repeater" for a different concept, which argues against adopting it here |
| 4 | "Item events" vs. "Element events" for user-triggered events | The "interaction" term is glossary-conflicted; what's the right plain-language name? | **"Item events"** (scoped to Repeater items, consistent with "Repeater item") vs. **"Element events"** (matches Velo API framing of `$w` as element selectors) — recommend "Item events" for panel copy, "event handler" for developer docs |

---

## Glossary Cross-Reference Summary

| Term | Glossary Status | Domain | Action |
|------|----------------|--------|--------|
| Repeater | ✅ Canonical | Velo | Keep; use lowercase in UX copy |
| Event handler | ✅ Canonical | Velo | Keep; use consistently |
| Event | ✅ Canonical | Velo | Keep |
| Interaction | ⚠️ Prohibited (Studio Editor / Harmony) | Studio Editor | Do not use in Studio 2 UI; developer docs (Velo context) may keep it |
| item (for grid children) | ❌ Prohibited | Studio Editor | Use "Cell" for grid children; "item" is fine for Repeater items |
| template | ✅ Canonical (other domains) | Get Paid | Acceptable; no Repeater-specific definition found |

---

## Sources

- **Competitor research**: `artifacts/discovery/competitor-research-repeater-event-handlers.md`
- **User Voice analysis**: `artifacts/discovery/user-voice-repeater-event-handlers.md`
- **Internal discovery**: `artifacts/discovery/internal-discovery-repeater-event-handlers.md`
- **Wix UX Writing Glossary**: Retrieved via API (`https://n8n-product.wixprod.net/webhook/bb0af908-14b5-43ec-bd62-ffb041e5eac3`) — VPN required
- **Velo API reference**: [dev.wix.com Repeater API](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater/introduction)

# User Voice Research: Studio 2 Editor — Repeater Event Handlers

**Issue types:** Missing Feature, Bug | **Signal Strength:** Moderate | **Confidence:** Medium

> ⚠️ Support ticket data (User Voice MCP) and Jira were unavailable during this session due to auth. Signal is based entirely on public channels: Wix Studio Community Forum, official Velo API docs, and developer blog posts. Findings likely undercount the true volume — treat mention counts as minimums.


---


## General Sentiment
**Mood:** Frustrated — Developers and Partners consistently hit unexpected silences (events that should fire don't) and scope leakage (events that fire affect the wrong item), with no clear signal from Wix about whether these are bugs or working-as-designed.


---


## The Opportunity
> Studio 2 has an opening to define a coherent, reliable event model for the Repeater as a first-class component — moving beyond the Velo scripting API's patchwork of lifecycle hooks and scoping workarounds, and giving developers a predictable contract for how Repeater events behave.

The CMS group is now owning Repeater as a component (vs. Velo group's API-first approach), which makes this the right moment to define what should be exposed in the Studio 2 editor panel rather than leaving it implicit in the API.


---


## Signal Overview

| Who | Intent | Gap | Signal |
| :--- | :--- | :--- | :--- |
| **Developers** | React when any item in a Repeater changes state (added, updated, removed) | `onItemReady` doesn't fire on data updates; `onItemUpdated` doesn't exist | Multiple Community Forum threads |
| **Developers and Partners** | Write event handlers scoped to the specific item the user interacted with | `event.context.itemId` returns wrong item; Wix's Sept 2024 update broke `$item` selector | 2+ Forum threads, including a post-update regression |
| **Developers and Partners** | Use visual event connections (click, hover) per Repeater item | Events fire across all items instead of the clicked one; `$w` vs `$item` scoping is non-obvious | Multiple Forum threads |
| **Studio Users and Partners** | Use hover/mouse effects on dataset-connected Repeaters | `onMouseIn`/`onMouseOut` silently stop working when Repeater is dataset-connected | 1 Forum thread |


---


## Findings

### Missing Feature

#### Full item lifecycle coverage (onItemUpdated missing)

**Intent:** Developers want to react when a Repeater item's data changes — not just when items are added or removed.
**Gap:** Only `onItemReady` (on creation) and `onItemRemoved` (on removal) exist. There is no `onItemUpdated` or equivalent. Setting `.data` on an existing Repeater item does not re-trigger any lifecycle event.
**Scale:** 3+ Community Forum threads | **Severity:** High

**Workaround:** Developers call `forEachItem()` or `forItems()` manually after setting `.data` — this is boilerplate every developer has to write themselves.

**User voice:**

> *"onItemReady not working on repeater update"* — Developer, Wix Studio Community Forum ([link](https://www.wix.com/velo/forum/coding-with-velo/onitemready-not-working-on-repeater-update))

---

> *"The callback is not triggered for existing items that are updated when you set the data property."* — Official Velo API Docs limitation note ([link](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater/on-item-ready))

---

**Suggested solution:** Define `onItemUpdated` as a first-class lifecycle event for Repeater in Studio 2, and expose it in the Properties & Events panel alongside `onItemReady` and `onItemRemoved`.

---

#### Mouse/hover events unavailable on dataset-connected Repeaters

**Intent:** Partners and Studio Users want to apply hover effects and mouse interactions on Repeaters that are connected to a dataset — the most common production use case.
**Gap:** `onMouseIn`/`onMouseOut` work in isolation but silently fail when the Repeater is dataset-connected.
**Scale:** 1 Forum thread (likely undercounted — this would show as a silent workaround, not a report) | **Severity:** Moderate

**Workaround:** None documented. Users either remove the data connection or remove the hover effect.

**User voice:**

> *"Hover event in repeater not working when its connected to a Database"* — Studio User, Wix Studio Community Forum ([link](https://forum.wixstudio.com/t/hover-event-in-repeater-not-working-when-its-connected-to-a-database/3515))

---

**Suggested solution:** Audit whether `onMouseIn`/`onMouseOut` (and equivalent hover events) should be exposed at the Repeater level in Studio 2's event panel, and whether the dataset-connection breakage is a platform bug to fix or a conscious scope decision to document.


---


### Bug

#### Event context returns wrong item (post-Sept 2024 regression)

**Intent:** Developers want `event.context.itemId` to reliably identify which Repeater item triggered an event.
**Gap:** After Wix's Sept 2024 legacy→dynamic event handler update, `$item` inside a Repeater event handler always returns data from the first item, regardless of which item was interacted with.
**Scale:** 2 Forum threads (including a post-update regression report) | **Severity:** High

**Workaround:** Use `$w.at(event.context)` to get a correctly scoped selector. This works but is not documented prominently and breaks the expected `$item` pattern.

**User voice:**

> *"After the recent update for event handlers making them legacy and dynamic… `$item('#examsessionsDB').getCurrentItem().fieldName` always returns the value of the first item of repeater."* — Developer, Wix Studio Community Forum, Sept 2024 ([link](https://forum.wixstudio.com/t/event-handlers-legacy-and-dynamic-update/63147))

---

> *"The event context of the repeater is not returning the correct Item Id."* — Developer, Wix Studio Community Forum ([link](https://forum.wixstudio.com/t/solved-the-event-context-of-the-repeater-is-not-returning-the-correct-item-id/35174))

---

**Suggested solution:** Before finalizing Studio 2's event handler contract for Repeater, verify with the Velo/platform team whether this regression was fixed or is still live. If still live, the Studio 2 event model needs to either work around it or block on the fix.

#### Events firing across all items instead of the clicked item

**Intent:** Developers want click and interaction events to apply only to the specific Repeater item the user acted on.
**Gap:** Using `$w('#element')` inside a Repeater event handler selects the element across all items. The correct pattern (`$item`) is non-obvious and poorly surfaced in documentation.
**Scale:** 2+ Forum threads | **Severity:** Moderate

**Workaround:** Use `$item` selector (scoped to current item) or `forItems([event.context.itemId], ...)` to isolate the interaction.

**User voice:**

> *"Event in Repeater Occurring Across All Items Instead of Targeted Item Only"* — Developer, Wix Studio Community Forum ([link](https://forum.wixstudio.com/t/solved-event-in-repeater-occurring-across-all-items-instead-of-targeted-item-only/41788))

---

**Suggested solution:** Studio 2's code generation for Repeater event handlers should default to the `$item`-scoped pattern — not `$w` — so the common mistake is impossible by default.


---


## Silent Churn

These findings come entirely from developers and partners who bothered to post on a community forum — a small fraction of those who hit the issue. Users who quietly abandoned their Repeater implementation, reverted to non-interactive layouts, or switched platforms are not represented. The hover/mouse issue in particular is almost certainly undercounted: it manifests as a silent failure with no error, making it far more likely to be dropped than reported.


---


## Recommended Manual Checks

| Source | Search for | Why |
| :--- | :--- | :--- |
| **Wix Partner Forum** | `repeater event handler`, `repeater onItemReady`, `repeater click item` | Partners build complex Repeater-driven UIs and are more likely to document edge cases in depth |
| **Wix Roadmap / Wishlist** | `repeater event`, `onItemUpdated`, `repeater hover` | Would reveal whether `onItemUpdated` has been formally requested and how many votes it has |
| **Internal Velo/Platform Slack or spec docs** | Sept 2024 event handler migration details | Critical to understand whether the `$item` scoping regression was intentional or a bug, and whether it's been addressed |


---


## Next Steps

* **Data:** Query how many active Studio sites use a Repeater with at least one event handler wired — this sizes the audience affected by the scoping bugs and gaps.
* **Product:** Map current Velo Repeater API surface (`onItemReady`, `onItemRemoved`, `forEachItem`, `forItems`) against what Studio 2's Properties & Events panel should expose. Specifically decide: (1) add `onItemUpdated`, (2) surface or deprecate `forEachItem`/`forItems` in the panel, (3) handle hover/mouse event availability for dataset-connected Repeaters.
* **Validation:** Sync with Velo/platform team on the Sept 2024 `$item` regression status before finalizing the Studio 2 event contract.

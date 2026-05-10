# User Voice — Repeater in Repeater (Studio 2 CMS)

**Project:** Repeater in Repeater (Studio 2 CMS)
**Phase:** Discovery — User Voice
**Date:** 2026-05-06
**Sources:** User Voice MCP (support tickets + chat + phone), Wix Studio Community Forum, Wix Help Center request page. Jira (UPI / EDITOR / CMS) was unavailable for this pass — auth needed.

> **Issue type:** Missing Feature (with strong Usability spillover when builders try to fake it).

---

## Signal Overview

| | |
|---|---|
| **Who** | Self-Creators and Studio Users / Partners building data-rich sites — catalogs, directories, programs, member areas, restaurants, real-estate listings. Builders coming from Webflow / Bubble assume nested repeaters work natively. |
| **Intent** | Render a list of children inside each parent item — products under a category, episodes under a season, dishes under a meal, ingredients under a recipe, lessons under a course, contacts under a property. |
| **Gap** | Studio 2 (and the legacy Editor) does not allow attaching a Repeater inside a Repeater item bound to that parent item's array / multi-reference field. Builders fall back to multi-dataset hacks, multiple parallel Repeaters with hardcoded filters, or Velo code — all fragile. Wix Blocks already supports nested dynamic Repeaters (via widget prop-drilling and `include()`), so the platform recognizes the need but hasn't surfaced it in the canvas binding model. |
| **Signal strength** | **Strong / High confidence.** Triangulated across support tickets (155+ on related multi-reference issues, 76+ on shared-dataset filter issues, 119 on layout fallout from "multiple repeaters instead of a unified structure"), an officially-acknowledged Help Center request page ("CMS Request: Attaching a Repeater onto Another Repeater"), multiple long-running Wix Studio Community threads, and parallel Velo forum threads. |

---

## Findings

### 1. The "I want a list of related things per item" pattern is the real ask

Users describe the intent in the same shape across very different domains: each parent row has its own short list of children, and they want that child list to render inline with the parent. They do not say "nested repeater" — they say "list under each item," "related content for this row," "dishes for this meal," "courses for this speaker." The naming on the Help Center page ("Attaching a Repeater onto Another Repeater") matches the literal canvas action they expect to perform.

**Why it matters:** This is the dominant real-world data shape for content sites — categories→items, parents→children, owners→assets. The current product forces them to flatten, duplicate, or code their way out of it.

**Quotes** (`---` between each):

**The course site builder filtering by speaker**
*"I have a reference 'Speaker Courses' in the 'Offerings' collection, which should filter specific speaker's courses to be displayed in the repeater on a dynamic page, but it doesn't work. I have checked the reference field setup, dataset connections, and filters, but the problem persists."* — Self-Creator, Support ticket (Chat), 2025-11-05

---

**The plant catalog with associated planets**
*"A plant can be associated with one or more planets... I would like to set up this filter element, this multi-choice, so that when they select Mars and Mercury, all the plants that are associated with Mercury or Mars in the multi-reference appear. And I haven't been able to do that before."* — Self-Creator, Support ticket (Phone, German), 2025-09-05

---

**The travel site that wants related packages on each item page**
*"I want to add a section below the item page for relative items."* — Self-Creator, Support ticket (Chat), 2025-03-10

---

### 2. The "multiple repeaters as a workaround" pattern is the strongest evidence of unmet need

Users repeatedly try to express nesting by placing several Repeaters on the same page, each bound to the same dataset with a different filter, expecting them to behave independently. They cannot — the dataset is shared, filters cascade, and layout drifts. The internal support label "**inconsistent spacing in repeater elements**" (119 highlights) literally calls this out: *"caused by improper stacking, alignment, proportional scaling, cascading rules, and the use of multiple repeaters instead of a unified structure."* That is the team naming the workaround.

**Why it matters:** When users invent the same hack independently across sectors (events, memberships, retreats, gold/silver tiers), the missing primitive is real. Today they fight the dataset and the layout system simultaneously.

**Quotes:**

**The events site trying to split intensives from retreats**
*"In the retreat section, I have a repeater below that and I have a filter on that repeater for items that are part of the reset series. And then below that, I have a second repeater and I'm trying to display the intensive items. But every time I change the filter on one, it changes the filter for the other one."* — Self-Creator, Support ticket (Phone), 2026-01-14

---

**The membership site trying to render gold / silver / bronze tiers**
*"I am trying to organize the listings so that the first, like, gold membership, is the first one and then the second section is silver. But the problem is, when I try to update the data set with the settings to be like, this one should have these listings, this one should not. It then applies that formula to all the different repeaters."* — Self-Creator, Support ticket (Phone), 2026-03-10

---

**The page-duplication workaround that breaks**
*"So I've created a repeater that pulls information from a database in the CMS. And I've copied the page with the repeater... but in the second page, the copied page, I wanted to have the repeater three times, but presenting three different filters. But every time I change the filter, it changes the filter in all of them, so they all look the same."* — Self-Creator, Support ticket (Phone), 2026-03-11

---

### 3. The platform partly admits it: Help Center, Studio Community, Velo forum

The need is documented in three Wix-owned surfaces — but only as a request, never as a delivered feature.

- **Help Center** has a dedicated request page titled *"CMS Request: Attaching a Repeater onto Another Repeater."* The framing: *"This feature would allow you to group items within a repeater based on a reference field. For example, you would be able to group dishes within the main repeater by meal."* — That is the canonical user intent in Wix's own words.
- **Wix Studio Community Forum** has at least four active threads (some marked "Solved" via workaround): "Looking For Nested Repeater Work-Around," "Workaround for nested repeaters needs," "Nested Repeater inside repeater," "Nested Repeater," "How to add a dynamic repeater to collection-based dynamic pages."
- **Velo Forum + Wix Studio Forum** — multiple threads on "Display multi-reference field in repeater" and "Display multiple multi-reference fields in a repeater, with title and link" — same root need from the developer-leaning segment.
- **Wix Blocks already ships it** for app developers via "Nest Dynamic Repeaters" using widget prop-drilling and `include()`. The pattern is: outer Repeater binds to parent collection, child Repeater is wrapped in a widget that receives the parent item's row through a prop, and the dev calls `include()` on the cross-collection field. Useful proof of concept — but it requires code, lives in Blocks not Studio editor, and doesn't reach the Self-Creator at all.

**Why it matters:** The need is recognized at the help-content level and partly solved for the Blocks/code path, but the Self-Creator and Studio canvas user have nothing. This is a coverage gap, not a discovery gap.

---

### 4. Adjacent issues that look like they belong here (review before scoping)

These showed up in the search and may or may not be in scope for the nested-Repeater PRD. Recommend treating them as informers, not deliverables:

| Adjacent issue | Mentions | How it relates |
|---|---|---|
| Multi-reference field not displaying dynamic data in repeater | 108 | The data plumbing under nested repeater. Fixing nested without fixing this leaves users stuck. |
| Repeaters cannot have independent filters due to shared dataset | 76 | A separate primitive (per-instance dataset state) but often the wrong tool users reach for to fake nesting. |
| Conditional functionality in repeaters requires Velo coding | 15 | If nested-repeater binding stays code-gated, this complaint scales up. |
| Selection tags fail to display dynamic CMS blog categories | 6 | Confirms the category/tag→child-list mental model. |
| Missing functionality to reorder repeater items and columns | 5 | Once nested ships, sorting per-nest will follow immediately. |

---

## The Sharp Take

The **nested-Repeater shape is not a niche power-user request** — it's the default for every content site that has parents and children, and Self-Creators reach for it constantly. Today the platform forces three escape hatches (multi-dataset filter hacks, page duplication, Velo / Blocks code) and **all three break in predictable ways the support team is already labeling internally** ("multiple repeaters instead of a unified structure"). Wix Blocks proves the engine can do it; Studio 2 needs to bring it to the canvas-binding model so a Self-Creator never has to learn `include()`, datasets, or widget props to render dishes under meals.

---

## Opportunity

A first-class nested-Repeater binding flow in Studio 2 — where the outer Repeater is bound to a parent collection, and the inner Repeater's `Items` picker offers the parent item's array / multi-reference fields with a clear `This item` scope badge — would:

- Replace three documented workarounds (multi-dataset, page-copy, Velo) with one canvas action.
- Reduce a measurable category of Customer Care load (the multi-reference + shared-dataset clusters above).
- Close a Webflow / Framer / Bubble parity gap that currently shows up in switching-intent threads.
- Unlock a class of templates (catalog with sub-collections, directories, restaurants, learning, real estate) that are awkward today.

The PRD should specify how the inner Repeater's binding picker presents `This item` scope, how depth is bounded (2 levels is the realistic ceiling for now), and how disconnect cascades when the parent's `Items` is changed or removed.

---

## Recommended Next Steps (product, not UX)

1. **Confirm scope with engineering** that the runtime can drive an inner Repeater off a parent item row without dataset duplication — Blocks already does this; Studio's binding platform should reuse that primitive.
2. **Audit the Help Center "CMS Request" page volume** (signups / votes if tracked) for sizing. Surface the Wix Studio Community thread metrics where possible.
3. **Design the binding picker for `This item` scope** as a follow-up to the existing `This repeater` work — same surface, one additional scope.
4. **Decide the depth limit** (2 levels recommended for v1) and call it out explicitly in the PRD.
5. **Defer the AI flows** to a separate section as the blank-Repeater PRD already does.

---

## Recommended Manual Checks (auth-gated channels)

- **Wix Roadmap / Wishlist** — search "nested repeater," "repeater in repeater," "repeater within repeater," "attach repeater."
- **Wix Partner Forum** — same terms; agency builders are the heaviest hitters here.
- **Jira UPI / EDITOR / CMS / STUDIO projects** — re-run after Jira auth is restored. Specifically search:
  - `text ~ "nested repeater" OR text ~ "repeater in repeater" OR text ~ "repeater inside repeater"` in `UPI` (CCFRs).
  - `text ~ "nest repeater" OR text ~ "child repeater" OR text ~ "inner repeater"` in `EDITOR`, `CMS`, `STUDIO`.

---

## Sources

- [CMS Request: Attaching a Repeater onto Another Repeater (Wix Help Center)](https://support.wix.com/en/article/cms-request-attaching-a-repeater-onto-another-repeater)
- [Looking For Nested Repeater Work-Around (Wix Studio Forum)](https://forum.wixstudio.com/t/solved-looking-for-nested-repeater-work-around/49870)
- [Workaround for nested repeaters needs (Wix Studio Forum)](https://forum.wixstudio.com/t/workaround-for-nested-repeaters-needs/35549)
- [Nested Repeater inside repeater (Wix Studio Forum)](https://forum.wixstudio.com/t/nested-repeater-inside-repeater/14467)
- [Nested Repeater (Wix Studio Forum)](https://forum.wixstudio.com/t/nested-repeater/47869)
- [How to add a dynamic repeater to collection-based dynamic pages (Wix Studio Forum)](https://forum.wixstudio.com/t/how-to-add-a-dynamic-repeater-to-collection-based-dynamic-pages/55743)
- [Display multi-reference field in repeater (Wix Studio Forum)](https://forum.wixstudio.com/t/display-multi-reference-field-in-repeater/9636)
- [Display multiple multi-reference fields in a repeater (Wix Studio Forum)](https://forum.wixstudio.com/t/display-multiple-multi-reference-fields-in-a-repeater-with-title-and-link/8782)
- [Nest Dynamic Repeaters — Wix Blocks dev docs](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/nest-dynamic-repeaters)
- [Dynamic Repeaters in Blocks](https://dev.wix.com/docs/build-apps/develop-your-app/frameworks/wix-blocks/cms-collections-in-blocks/dynamic-repeaters-in-blocks)
- [CMS: Displaying Collection Content in a Repeater (Help Center)](https://support.wix.com/en/article/cms-displaying-dynamic-content-in-a-repeater)
- [CMS: Using 'Multi-Reference' Fields to Display Content from Multiple Collections (Help Center)](https://support.wix.com/en/article/cms-creating-multi-reference-fields)
- [Repeater Introduction — Velo API docs](https://dev.wix.com/docs/velo/velo-only-apis/$w/repeater/introduction)

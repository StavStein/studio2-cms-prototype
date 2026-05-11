# Repeater Event Handlers — Research Brief

---

**Coverage assessment:**
- Problem statement: Partial — confirmed via public community signal; no internal support ticket data available
- Target audience: Partial — Developers and Partners confirmed; no KB-validated segment sizes
- Expected business outcome: Partial — Developer/Partner adoption of Studio 2 + fewer support escalations; mechanism is clear, magnitude unknown
- Competitive landscape: Not explored — will be covered in competitor research phase
- Dimensions covered: 5 of 7 — Target audience, Problem validation, Business impact, Adoption/usability, Strategic alignment
- Dimensions not covered: Ecosystem/cross-domain dependencies (Velo platform, dataset layer, Properties & Events panel not mapped); Competitive landscape
- Recommended before handoff: Complete competitor research phase; sync with Velo/platform team on Sept 2024 `$item` regression status

---

**Problem statement:**

The Repeater component in Wix Studio currently exposes only two lifecycle event hooks — `onItemReady` (fires when items are created) and `onItemRemoved` (fires when items are removed). There is no `onItemUpdated` event, which means developers cannot react programmatically when a Repeater item's data changes in place — a fundamental gap for dynamic data-driven interfaces. Additionally, event scoping is unreliable: `event.context.itemId` returns incorrect item IDs in certain conditions, and a platform update in September 2024 broke the `$item` selector inside event handlers, causing it to always return data from the first item regardless of which item was interacted with. Mouse/interaction events (`onMouseIn`, `onMouseOut`) silently fail when the Repeater is connected to a dataset, which is the most common production use case. The CMS group is now owning the Repeater component definition for Studio 2 — previously managed by the Velo group as an API-first feature — and this is the right moment to define a coherent event contract that treats Repeater as a first-class component with a reliable, predictable behavior surface.

**Target audience:**

- **Primary user:** Developers and Partners (agencies/freelancers) writing Velo/JS code in Wix Studio who build interactive Repeater-based interfaces
- **User-of-User:** N/A — this is an editor/developer feature; the UoU impact is the site visitor experience, but it's indirect and not the subject of this analysis
- **Target segment:** Active Studio developers and Studio-registered partner accounts; specifically those building data-connected or interactive Repeater layouts
- **Other segments considered:** Self-Creators who use visual event connections (Properties & Events panel) without writing custom code — a secondary segment that may benefit from improved panel event coverage

**Expected business outcome:**

A well-defined Repeater event model in Studio 2 improves **Developer and Partner adoption of Studio 2** — developers encountering fewer dead-ends means less friction in the migration from Classic editor. Fewer event-model surprises also reduce **support escalations** for Repeater-related issues, lowering the support cost associated with the Repeater. The strategic lever: as Partners and Developers adopt Studio 2 (driven by a more reliable component API), **Studio Accounts and Studio Premiums** grow, which directly contributes to the Partner segment of Collections.

**Domain context:**

No domain KB was available during this session (OAuth auth failure). Context is based on the project brief, user voice research, and conversation.

- Repeater component is part of the Studio 2 Editor, owned by the CMS group going forward
- Prior Velo API surface: `onItemReady`, `onItemRemoved`, `forEachItem`, `forItems`, `data` property
- Known platform dependency: Sept 2024 legacy→dynamic event handler migration — status of the `$item` scoping regression is unconfirmed
- Relevant audience tiers (not KB-validated): Active Studio developer accounts; Partner accounts (agencies/freelancers) managing multiple client sites in Studio

**Competitive landscape:**

Not explored in this session. Will be covered by the competitor research phase.

---

**Key decisions to make:**

*How to build it:*
- Which lifecycle events to expose in the Properties & Events panel for Repeater (must-resolve: `onItemReady`, `onItemRemoved` already exist — should `onItemUpdated` be added?) — must-resolve
- Whether to fix or formally document the dataset-connection breakage for hover/mouse events in Studio 2 — must-resolve
- Whether to surface `forEachItem`/`forItems` as panel-level interactions or keep them code-only — should-resolve
- Whether the Sept 2024 `$item` regression is fully fixed before Studio 2 ships (dependency from Velo/platform team) — must-resolve (blocking)
- Which event handler model to generate by default in code stubs (should generate `$item`-scoped pattern, not `$w`) — should-resolve

**Questions to investigate:**

*How to build it questions:*

- **Q1 — Who is building with Repeater + event handlers in Studio today, and at what scale?**
  This defines the size of the audience whose day-to-day will be directly affected by the Studio 2 event model spec.
  - **Metrics:**
    - **Metric:** Count of distinct developer/partner accounts with at least 1 published Studio page containing Velo code that references a Repeater component
      - **Population:** Active Studio accounts (accounts with at least 1 published site in Studio in last 90 days)
      - **Cohort filter:** Accounts where at least 1 published page includes Velo code referencing `$w('#').onItemReady` or `$w('#').onItemRemoved` or a Repeater `data` assignment
      - **Timeframe:** Last 90 days rolling
      - **Logic:** Count of distinct account IDs meeting filter criteria; breakdowns by account type (Partner vs. Developer vs. Self-Creator)
      - **KB reference:** None available
    - **Metric:** Count of sites (msid) with at least 1 Repeater component with a wired event handler (any type)
      - **Population:** Published Studio sites with Repeater component instances
      - **Cohort filter:** Sites published at least once, with at least 1 Repeater element on any page and at least 1 event handler wired to that Repeater
      - **Timeframe:** Current snapshot
      - **Logic:** Count distinct msids; breakdowns by editor type (Studio vs. Classic) to show migration state
      - **KB reference:** None available
    - **Metric:** Studio vs. Classic split for sites with Repeater — trend over last 12 months
      - **Population:** Sites with Repeater component created in each monthly cohort
      - **Cohort filter:** Sites created each calendar month over the last 12 months that include a Repeater element
      - **Timeframe:** Monthly cohorts, last 12 months
      - **Logic:** For each monthly cohort: count of Repeater sites created in Studio vs. Classic; compute share of Studio over time to show adoption trajectory
      - **KB reference:** None available
  - **Breakdowns:** Partner accounts vs. Developer accounts vs. Self-Creator accounts; Studio vs. Classic editor; country (top 5 markets)

- **Q2 — Which event handler types are developers actually using on Repeater today?**
  Directly informs v1 event list: events with high usage must be in the spec; rare events can be deferred.
  - **Metrics:**
    - **Metric:** Distribution of Repeater event handler types in published Velo code
      - **Population:** Published pages with Velo code that includes Repeater event handler calls
      - **Cohort filter:** Pages published in last 90 days, with at least 1 Repeater element and at least 1 event handler call in the page's Velo code
      - **Timeframe:** Last 90 days rolling
      - **Logic:** For each event handler type (`onItemReady`, `onItemRemoved`, `onClick` on child elements, `onMouseIn`, `onMouseOut`, custom), count distinct pages where that type appears; express as % of all Repeater-with-event pages. Show P50/P75 (median and upper quartile) pages per account using each type.
      - **KB reference:** None available
    - **Metric:** Co-occurrence of event handler types per page
      - **Population:** Same as above
      - **Cohort filter:** Pages with 2+ distinct event handler types wired to Repeater
      - **Timeframe:** Last 90 days rolling
      - **Logic:** Count pages by event handler type combination (e.g., `onItemReady` + `onClick`, `onItemReady` + `onMouseIn`) to identify which combinations developers use together — informs whether these must be consistent in Studio 2
      - **KB reference:** None available
  - **Breakdowns:** Partner accounts vs. Developer accounts; Studio vs. Classic

- **Q3 — What share of Repeater instances are dataset-connected vs. using static/code-assigned data?**
  Determines the scope of fixing hover/mouse events for dataset-connected Repeaters — if 80%+ are dataset-connected, fixing this is a must-have, not optional.
  - **Metrics:**
    - **Metric:** Proportion of sites with Repeater + active dataset connection
      - **Population:** Published sites with at least 1 Repeater component
      - **Cohort filter:** Active sites (had at least 1 editor session or site visit in last 90 days) with Repeater
      - **Timeframe:** Current snapshot
      - **Logic:** Count of sites where Repeater is connected to a dataset / total sites with Repeater; present as percentage with P25/P50/P75 across accounts to show distribution
      - **KB reference:** None available
    - **Metric:** Dataset-connected Repeater sites with at least 1 mouse/hover event handler wired
      - **Population:** Published sites with Repeater + dataset connection (from above)
      - **Cohort filter:** Sites where the Repeater has `onMouseIn` or `onMouseOut` event handlers wired in any form (panel or code)
      - **Timeframe:** Current snapshot
      - **Logic:** Count of sites affected by the hover/dataset-connection bug; this is the direct impact audience for fixing or formally scoping that issue
      - **KB reference:** None available
  - **Breakdowns:** Studio vs. Classic; Partner vs. Developer accounts

- **Q4 — What is the adoption rate of Repeater with event handlers by new Studio developer accounts post-onboarding?**
  Baseline for measuring whether Studio 2's improved event model increases developer confidence in using Repeater for interactive use cases.
  - **Metrics:**
    - **Metric:** Funnel — new Studio accounts that create a Repeater → add event handler → publish, within 30 days of first Studio session
      - **Population:** Accounts that opened Studio for the first time in a given monthly cohort
      - **Cohort filter:** Accounts with first Studio session in each calendar month (monthly cohort anchor: `first_studio_session_date`)
      - **Timeframe:** Monthly cohorts over last 6 months; measure 30-day conversion window from cohort anchor
      - **Logic:** For each cohort: count of accounts that (1) created a Repeater, (2) wired at least 1 event handler to it, and (3) published the site — all within 30 days. Compute rate = (accounts reaching step 3) / (accounts reaching step 1). Track as a monthly trend to see whether the current model is improving or degrading developer onboarding to Repeater.
      - **KB reference:** None available
    - **Metric:** Time-to-first-event-handler for Repeater among new Studio developers
      - **Population:** New Studio accounts (first Studio session in last 6 months) who created a Repeater
      - **Cohort filter:** Accounts that created at least 1 Repeater in their first 30 days in Studio
      - **Timeframe:** Last 6 months
      - **Logic:** Median (P50) and P75 days from Repeater creation to first event handler wired on that Repeater; longer time-to-first-event suggests friction in the current model
      - **KB reference:** None available
  - **Breakdowns:** Partner vs. Developer accounts; Studio cohort month

---

**Open gaps:**

- **Sept 2024 `$item` regression status:** Whether this platform bug is fixed or still live is unknown from public sources. If still live, Studio 2's event model is building on a broken foundation. Must confirm with Velo/platform team before finalizing the spec.
- **Properties & Events panel scope:** What the panel currently shows for Repeater when selected in Studio editor is not fully documented. Unclear whether `onItemReady` and `onItemRemoved` already appear in the panel or are code-only. This affects spec scope.
- **`forEachItem` / `forItems` panel exposure:** No decision made yet on whether these utility methods should be surfaceable from the panel or remain code-only.
- **Competitive landscape:** Not explored. Competitor research phase will cover how other editors (Webflow, Framer, Builder.io) handle component event models.

**KB-anchored context:**
- Relevant existing KPIs: None available — KB was inaccessible during this session
- Relevant data tables: None identified — no analytical KB queried
- Key segments from KB: Not KB-validated. Segments used in this brief (Developer accounts, Partner accounts, Self-Creator accounts) are conversation-derived from the project brief.

**Go/no-go gate:** Not applicable — build decision is already made. This analysis is scoped to informing the spec (what to build, what to prioritize, how to scope v1).

**Dependencies:**
- Q3 (dataset-connected Repeater scope) should be run before finalizing whether hover/mouse event fix is a v1 requirement
- Q2 (event handler type distribution) directly informs the v1 event list and should be run before writing the spec
- Q4 (adoption funnel) establishes the baseline; compare against post-Studio 2 launch to measure spec quality impact

**Suggested next step:** This brief is structured for a query skill to pull data automatically. If that skill is available, run it next. Otherwise, hand this to your data team with Q2 and Q3 as the highest-priority queries.

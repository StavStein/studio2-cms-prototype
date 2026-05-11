<!-- creator-kit:artifact-map:start -->
<!-- WARNING: Creator Kit updates everything between these markers; add custom instructions outside this block. -->

# Creator Kit Artifact Map

purpose: project-local routing index for generated Creator Kit artifacts

creator_kit:
- install: `npx -y --registry "https://npm.dev.wixpress.com" @wix/creator-kit@latest install`
- navigator: `ck-use-creator-kit`
- help: `ck-help`
- status: `ck-status`
- extensions: optional project guidance in `.agents/skills/ext-ck` and `.agents/skills/ext-ck-*`; use `ck-ext-new`

rules:
- read this before loading artifacts
- load only paths needed for the current task
- after artifact create/rename/delete, update this block before final response
- artifact files win if this map is stale
- create or update an ADR for architecture-impacting or hard-to-reverse decisions
- this is an index, not a summary

pointers:
- artifacts: `artifacts/`
- context: `context/`
- decisions: `adr/`
- references: `missing`

artifacts:
- `artifacts/discovery/competitor-research-repeater-event-handlers.md` | discovery | `ck-research-competitors` | competitor research | inputs: project brief
- `artifacts/discovery/competitor-research-repeater-in-repeater.md` | discovery | `ck-research-competitors` | competitor research | inputs: project brief
- `artifacts/discovery/data-analysis-plan-repeater-event-handlers.md` | discovery | `ck-data-analysis-plan` | data analysis plan | inputs: project brief
- `artifacts/discovery/data-analysis-plan-repeater-in-repeater.md` | discovery | `ck-data-analysis-plan` | data analysis plan | inputs: project brief
- `artifacts/discovery/data-analysis-plan-rir-vertical-data.md` | discovery | `ck-data-analysis-plan` | data analysis plan | inputs: project brief
- `artifacts/discovery/data-analysis-repeater-in-repeater.md` | discovery | `ck-data-query` | data analysis | inputs: data analysis plan
- `artifacts/discovery/data-analysis-rir-vertical-data.md` | discovery | `ck-data-query` | data analysis | inputs: data-analysis-plan-rir-vertical-data.md
- `artifacts/discovery/internal-discovery-repeater-event-handlers.md` | discovery | `ck-research-wix-internal` | internal discovery | inputs: project brief
- `artifacts/discovery/internal-discovery-repeater-in-repeater.md` | discovery | `ck-research-wix-internal` | internal discovery | inputs: project brief
- `artifacts/discovery/research-summary-repeater-event-handlers.md` | discovery | `ck-research-summary` | discovery synthesis | inputs: discovery artifacts
- `artifacts/discovery/research-summary-repeater-in-repeater.md` | discovery | `ck-research-summary` | discovery synthesis | inputs: discovery artifacts
- `artifacts/discovery/terminology-research-repeater-event-handlers.md` | discovery | `ck-research-terminology` | terminology research | inputs: brief, discovery
- `artifacts/discovery/terminology-research-repeater-in-repeater.md` | discovery | `ck-research-terminology` | terminology research | inputs: brief, discovery
- `artifacts/discovery/user-voice-repeater-event-handlers.md` | discovery | `ck-research-user-voice` | user voice research | inputs: project brief
- `artifacts/discovery/user-voice-repeater-in-repeater.md` | discovery | `ck-research-user-voice` | user voice research | inputs: project brief
- `artifacts/project-brief-repeater-event-handlers.md` | discovery | `ck-new` | project brief | inputs: user intake
- `artifacts/project-brief-repeater-in-repeater.md` | discovery | `ck-new` | project brief | inputs: user intake
- `artifacts/project-brief-rir-vertical-data.md` | discovery | `ck-new` | project brief | inputs: user intake
- `artifacts/project-brief-sort-filter-behavior.md` | discovery | `ck-new` | project brief | inputs: user intake
- `artifacts/product/product-spec-repeater-event-handlers.md` | product | `ck-product-spec` | product spec | inputs: strategy, discovery
- `artifacts/product/product-spec-repeater-in-repeater.md` | product | `ck-product-spec` | product spec | inputs: strategy, discovery
- `artifacts/product/product-spec-repeater-pagination.md` | product | `ck-product-spec` | product spec | inputs: strategy, discovery
- `artifacts/research-summary-repeater-event-handlers/index.html` | research-summary-repeater-event-handlers | unknown | index | inputs: unknown
<!-- creator-kit:artifact-map:end -->

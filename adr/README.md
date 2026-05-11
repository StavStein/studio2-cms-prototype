# ADR Directory

This directory stores Architecture Decision Records (ADRs) for architecture-impacting or hard-to-reverse project decisions.

## Naming Convention

- Use lowercase, dash-separated names.
- Use present-tense imperative verb phrases.
- Example: `adopt-adrs-for-project-decisions.md`

## ADR Required

Create an ADR when a change introduces one or more of the following:

- Cross-cutting workflow, runtime, or architecture behavior
- Validation or enforcement policy that changes how contributors or agents work
- Project conventions that are expensive or annoying to reverse later

## ADR Not Required

An ADR is usually not needed for:

- Session artifacts, working notes, or temporary exploration
- Small bug fixes or local refactors
- Minor implementation details that do not change project-level behavior

## Minimum Structure

Each ADR should include:

- `Date`
- `Context`
- `Decision`
- `Consequences`
- `Alternatives Considered` when meaningful alternatives were actually evaluated

Do not add a `Status` field. ADR approval happens through review and merge.

## Superseding Rule

Do not rewrite old ADRs. When a decision changes, add a new ADR and mark the previous ADR as superseded.

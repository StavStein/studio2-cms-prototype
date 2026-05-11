# adopt-adrs-for-project-decisions

- Date: 2026-05-10

## Context

This project needs one committed, skimmable surface for architecture-impacting or hard-to-reverse decisions about implementation direction, workflow policy, and contributor expectations.

Without that surface, rationale drifts into chat history, PR bodies, review threads, and scattered notes. The project brief explains the starting point, but it should not become the durable record for decisions made later.

## Decision

We will use Architecture Decision Records in `adr/` for architecture-impacting or hard-to-reverse project decisions.

ADRs are a durable record, not a working scratchpad. Temporary exploration and Creator Kit session outputs stay in `artifacts/`; creator-provided reference material stays in `context/`.

## Alternatives Considered

- Keep rationale only in the project brief: rejected because the brief is initial context, not a running decision log.
- Keep relying on PR descriptions and review threads: rejected because they are weak long-term lookup surfaces.
- Keep plans or working notes as the default durable surface: rejected because they blur accepted decisions with in-flight exploration.

## Consequences

- Positive: contributors and agents get a stable place to recover project decisions.
- Positive: `artifacts/` can stay focused on generated workflow outputs.
- Negative: contributors must decide when a change deserves an ADR.
- Follow-up: migrate older decision notes only when there is a concrete reason.

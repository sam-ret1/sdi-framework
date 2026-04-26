# IMPLEMENTATION_PLAN Template (Core)

This is the most prescriptive artifact. The coding agent consumes it directly. Ambiguity here causes rework; over-specification here is fine.

This template covers **universal sections**. Type-specific sections (database changes for tipos com DB, API contracts for HTTP services, scraping passes for data pipelines, prompt/tool design for agents) live in `project-types/{type}/` and are inserted at the marked locations below.

## Naming convention

The framework treats `IMPLEMENTATION_PLAN_*.md` uniformly. Use whichever naming fits the work item:

- **`IMPLEMENTATION_PLAN_PHASE_N.md`** — for discrete numbered phases (greenfield projects following a linear ROADMAP, or structured migrations). Examples: `PHASE_1`, `PHASE_2`.
- **`IMPLEMENTATION_PLAN_<slug>.md`** — for free-form work in ongoing projects (features, bugfixes, maintenance batches, performance passes). Examples: `IMPLEMENTATION_PLAN_billing-portal.md`, `IMPLEMENTATION_PLAN_q1-perf-pass.md`, `IMPLEMENTATION_PLAN_billing-fix.md`.

Throughout this template, sections referencing "Phase N" apply equally when the plan is named with a slug — substitute the slug as the work-item identifier. Headers below show "Phase N" as the dominant example; replace mentally if your plan uses a slug.

Target length: 400–600 lines (longer for complex phases).

## Structure

```markdown
# Implementation Plan — Phase N: [Phase Title]
# (or: Implementation Plan — <slug>: [Title] — for free-form work)

> Detailed spec for this work item. Companion to PRD.md, ARCHITECTURE.md, and PROJECT_STRUCTURE.md. This file is intentionally prescriptive where ambiguity would cause rework.
>
> **Revision note (rN):** [only present after implementation-time audits reshape the plan. Summarize what changed vs previous revision.]

## 0. Pre-requisites

- [Previous phase N-1 completed]
- [Required env vars / external accounts / setup steps]

## 1. Scope

[Brief description of what this phase produces.]

In scope:
- [bullets]

Out of scope for this phase:
- [bullets — explicit deferrals]

## 1.5 Runtime Dependencies to Install

- `package-name` — purpose (why chosen)
- ...

Explicitly NOT installing:
- `deferred-package` — deferred per DECISIONS.md #N

## 2. [Type-specific changes]

[Insert type-specific implementation sections here. Examples by type:
 - web-saas, api-service: Database changes (schema, RLS, migrations) + API Contracts (endpoints, status codes, auth)
 - landing-page: Page structure, content sources, SEO/meta, form integrations
 - dashboard: Data sources, queries, refresh strategy, charts
 - mobile: Screens, navigation, native integrations, offline behavior
 - data-pipeline: Sources, ingestion steps, transformations, output destinations, schedule
 - ai-agent: Prompts, tools, model config, eval suites, guardrails
 - automation-workflow: Triggers, steps, retries, idempotency keys, observability hooks

Load `project-types/{type}/` references for the canonical sub-section structure for the chosen type.]

## 3. [Domain-specific algorithm or verification]

[E.g. HMAC verification, mapping engine, state machine, retrieval ranking, scraping resilience — depth proportional to the subtlety of the thing.]

## 4. ...

[More sections as domain dictates: phone normalization, deduplication, rate limiting, content extraction, embedding indexing, etc.]

## 5. Rate Limiting / Resource Protection (or equivalent)

[If deferred, say so and document the stub pattern + env flag to activate real implementation later. Applies to APIs, scrapers, agent calls, and any system with cost/blast-radius concerns.]

## 6. UI Surfaces (if applicable)

[If this phase adds UI: pages/screens, who can access, key interactions. Skip entirely for non-UI projects.]

## 7. Background jobs / async work

[If this phase adds jobs: which event/trigger, what handler does, retry policy, idempotency.]

## 8. Tests (required before shipping this phase)

- Unit tests for: [list modules + edge cases]
- Integration tests for: [E2E flows]
- Manual smoke: [if relevant]
- Eval suite (AI projects): [reference golden set or eval config]

## 9. Observability

- [What to log, where, with what PII scrubbing rules]
- [Metrics or counters to surface, if applicable]

## 10. Acceptance Criteria

Phase N is done when:

1. [Testable criterion]
2. ...

## 11. Decisions Log (for DECISIONS.md)

Record at end of phase:

- [ ] [Decision X — why]
- [ ] ...

## 12. Known divergences from earlier docs

[Call out places where this plan knowingly disagrees with the higher-level PRD/ARCHITECTURE — and why. Usually resolved in ARCHITECTURE or ROADMAP updates at end of phase.]

- PROJECT_STRUCTURE.md says `[old path]`; repo uses `[new path]`. Use repo path. Update PROJECT_STRUCTURE at end of phase.

## How to feed this to the coding agent

[End-of-doc kickoff prompt. Keeps the plan self-contained — reader knows exactly how to start execution.]

> "Implement Phase N per `docs/IMPLEMENTATION_PLAN_PHASE_N.md` (rN). Follow `docs/PROJECT_STRUCTURE.md` for file locations and `AGENTS.md` for stack/conventions. When the plan disagrees with the actual repo, the repo wins — note the divergence in `DECISIONS.md` and proceed.
>
> Start by proposing:
> 1. [First deliverable — usually schema/foundation]
> 2. [Second — usually dependencies]
> 3. [Third — usually a narrow confirmation question about conventions]
>
> Stop there and wait for review. Do NOT start on [subsequent scope items] yet."
```

## Writing tips

- **Every "in scope" bullet corresponds to something specific elsewhere in the doc.** If §3 doesn't describe an algorithm for "ingest leads", don't list it in scope.
- **Type-specific schemas/contracts/specs (§2) should be close to production-ready.** Not final, but the structure should survive the implementation with only minor tweaks.
- **API contracts (when applicable) should include every status code.** If your contract table has 3 rows, you're missing error cases.
- **Acceptance criteria (§10) are a checklist the user can actually verify.** Avoid "X is stable" — use "X is covered by test Y" or "smoke test Z passes."
- **§11 Decisions Log is a checklist for end-of-phase housekeeping.** Each becomes a real DECISIONS.md entry at phase close.
- **§12 is where the plan is honest about its own limitations.** Usually empty at initial draft; populated during audit.

## Revision notes (r2, r3, ...)

When the coding agent audits this plan against the repo and finds divergences, the plan gets revised. The pattern:

```markdown
> **Revision note (r2):** audited against Phase N-1 repo state. 3 adjustments:
> 1. [Change — rationale]
> 2. [Change — rationale]
> 3. [Change — rationale]
>
> **General rule**: when this plan disagrees with actual repo conventions, the repo wins.
```

Revision notes stack at the top of the doc, newest first. Never delete prior revision notes — they're the history of the plan's evolution.

## Common failure modes

- **Too abstract.** If the plan reads like a PRD, it's not detailed enough. Include signature formats, status codes, prompt templates, schema fragments.
- **Too speculative.** Don't plan phases N+2 inside the Phase N plan. That's ROADMAP territory.
- **Missing out-of-scope.** §1 "out of scope for this phase" prevents scope creep. Don't skip it.
- **No acceptance criteria.** §10 is the contract for "when are we done." Don't leave vague.
- **Skipping the type-specific section.** §2 must be filled with the appropriate type appendix; otherwise the plan loses its prescriptive value.
- **Missing the kickoff prompt.** §"How to feed this to the coding agent" is valuable — users copy-paste it. If the plan doesn't have one, the user has to invent it, and they'll miss things.

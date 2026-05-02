# IMPLEMENTATION_PLAN Template (Core)

This is the most prescriptive artifact. The coding agent consumes it directly. Ambiguity here causes rework; over-specification here is fine.

This template covers **universal sections**. Type-specific sections (database changes for types with a database, API contracts for HTTP services, scraping passes for data pipelines, prompt/tool design for agents) live in `project-types/{type}/` and are inserted at the marked locations below.

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

Load only the context needed for this work item:
- Always start with this skill's `project-types/{type}/architecture-appendix.md` for type-specific implementation concerns.
- If the work item changes repo layout or conventions, read the target project's live `docs/PROJECT_STRUCTURE.md` and update the plan against that reality.
- If the work item adds or changes UI, read the target project's live `docs/DESIGN_SYSTEM.md` if it exists; if it is missing for a UI-bearing project, flag that as a pre-requisite/open question instead of inventing a design system from this skill.]

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

## 11. Implementation checkpoints (for sdi-mode)

This phase is implemented in checkpoints per the sdi-mode discipline. The canonical gate text below is mirrored from sdi-mode's `stop-and-review-patterns.md` so this plan is self-contained for review without sdi-mode installed. If the sdi-mode skill ever diverges from what's mirrored here, follow the skill — it carries the live discipline.

Skip checkpoints that don't apply (e.g. drop Checkpoint 4 if there is no UI). **Checkpoints 1 (Foundation) and 5 (Housekeeping) are non-negotiable.**

### Checkpoint 1 — Foundation **(user-gated)**

**Covers:** §0 Pre-requisites, §1 Scope (audit framing), §1.5 Runtime Dependencies, §2 schema/migrations/types (foundation only — no business logic).

**Standard gates (all ✓ before next checkpoint):**
- [ ] Audit report posted in canonical format
- [ ] All Blockers from the audit are either resolved or explicitly waived by the user
- [ ] All Open Questions from the audit are answered
- [ ] Each **material** plan-vs-repo Divergence has a corresponding `DECISIONS.md` entry; mechanical divergences are noted in the round report only
- [ ] Plan has a revision note (`rN`) summarizing audit changes (if any landed)
- [ ] User has given explicit go ("yes", "go", "proceed") — silence is **not** consent
- [ ] Today's `docs/memory/YYYY-MM-DD.md` entry mentions this checkpoint passing

**Phase-specific gates (optional — list only items unique to this phase):**
- [ ] [optional, e.g. "Migration 0008_billing_tables applied to local DB"]

### Checkpoint 2 — Core domain logic **(auto-review eligible)**

**Covers:** §3 Algorithm, §4+ pure logic sections, §8 unit tests for above.

**Standard gates:**
- [ ] All pure functions in scope for this checkpoint are implemented
- [ ] Unit tests cover the edge cases called out in the plan (count vs plan list)
- [ ] All unit tests pass (real count from the runner, not approximation)
- [ ] No TODO/FIXME left in the code without a corresponding `DECISIONS.md` or memory entry
- [ ] Round report posted in canonical format
- [ ] Auto-review (default) — reviewer ensemble returned merged PASS with all gates ✓ **OR** user opted out of auto-review and gave explicit go to move into integrations (see `auto-review-mode.md`)
- [ ] Today's `docs/memory/YYYY-MM-DD.md` entry summarizes the round

**Phase-specific gates:**
- [ ] [optional]

### Checkpoint 3 — Wire up integrations **(auto-review eligible)**

**Covers:** §2 endpoints/handlers, §5 Rate Limiting, §7 Background jobs, §9 Observability, §8 integration tests.

**Standard gates:**
- [ ] All endpoints/handlers/workers in scope for this checkpoint exist and respond
- [ ] Integration tests cover the canonical happy path + at least one failure mode per surface
- [ ] All integration tests pass against a real local instance (not mocks-only)
- [ ] Observability is wired (logs, metrics, tracing) per `ARCHITECTURE.md` requirements
- [ ] Manual smoke command attempted (curl/httpie/CLI) and result documented
- [ ] Round report posted
- [ ] Auto-review (default) — reviewer ensemble returned merged PASS with all gates ✓ **OR** user opted out and gave explicit go (see `auto-review-mode.md`)

**Phase-specific gates:**
- [ ] [optional]

### Checkpoint 4 — UI (skip if no UI in §6) **(auto-review eligible)**

**Covers:** §6 UI Surfaces.

**Standard gates:**
- [ ] Every page/screen in scope is reachable from navigation
- [ ] Forms validate per the schemas defined in Checkpoint 2
- [ ] Loading, empty, and error states exist for each data-driven surface
- [ ] Manual walkthrough of the primary flow completed; screenshots in the round report
- [ ] No console errors or accessibility warnings on the primary flow
- [ ] Round report posted
- [ ] Auto-review (default) — reviewer ensemble returned merged PASS with all gates ✓ **OR** user opted out and gave explicit go (see `auto-review-mode.md`)

**Phase-specific gates:**
- [ ] [optional]

### Checkpoint 5 — Housekeeping **(user-gated)**

**Covers:** §10 Acceptance Criteria mapping, §12 Decisions Log materialization, §13 Known divergences resolution, doc updates (`PROJECT_STRUCTURE`, `AGENTS.md`).

**Standard gates:**
- [ ] Every acceptance criterion in the current `IMPLEMENTATION_PLAN_*.md` §Acceptance Criteria has linked evidence
- [ ] `PROJECT_STRUCTURE.md` reflects the actual repo (no documented paths missing in code, no code paths missing in doc)
- [ ] `AGENTS.md` updates proposed and approved by user
- [ ] `DESIGN_SYSTEM.md` audit complete (UI types only) — tokens documented match tokens in code
- [ ] `DECISIONS.md` end-of-phase sweep done — no orphan/contradictory entries
- [ ] All revision notes on the plan reference resolved changes
- [ ] Lint passes
- [ ] Typecheck passes
- [ ] All unit + integration test suites pass
- [ ] Manual smoke test of the main acceptance criterion completed live and documented
- [ ] Phase tracker in `AGENTS.md` updated to ✓ with date
- [ ] Today's `docs/memory/YYYY-MM-DD.md` entry marks the phase as closed

**Phase-specific gates:**
- [ ] [optional]

## 12. Decisions Log (for DECISIONS.md)

Record at end of phase:

- [ ] [Decision X — why]
- [ ] ...

## 13. Known divergences from earlier docs

[Call out places where this plan knowingly disagrees with the higher-level PRD/ARCHITECTURE — and why. Usually resolved in ARCHITECTURE or ROADMAP updates at end of phase.]

- PROJECT_STRUCTURE.md says `[old path]`; repo uses `[new path]`. Use repo path. Update PROJECT_STRUCTURE at end of phase.

## How to feed this to the coding agent

[End-of-doc kickoff prompt. Keeps the plan self-contained — reader knows exactly how to start execution.]

> "Implement Phase N per `docs/IMPLEMENTATION_PLAN_PHASE_N.md` (rN). Follow `docs/PROJECT_STRUCTURE.md` for file locations and `AGENTS.md` for stack/conventions. When the plan disagrees with the actual repo, the repo wins — note the divergence in `DECISIONS.md` and proceed.
>
> Implement in checkpoints per §11 of the plan; standard gates for each checkpoint are inlined there. Start by proposing:
> 1. [First deliverable — usually Checkpoint 1 Foundation: schema + deps + audit]
> 2. [Second — Checkpoint 1 audit findings]
> 3. [Third — narrow confirmation question about conventions]
>
> Stop there and wait for review. Do NOT start on Checkpoint 2 (Core domain logic) yet."
```

## Writing tips

- **Every "in scope" bullet corresponds to something specific elsewhere in the doc.** If §3 doesn't describe an algorithm for "ingest leads", don't list it in scope.
- **Type-specific schemas/contracts/specs (§2) should be close to production-ready.** Not final, but the structure should survive the implementation with only minor tweaks.
- **API contracts (when applicable) should include every status code.** If your contract table has 3 rows, you're missing error cases.
- **Acceptance criteria (§10) are a checklist the user can actually verify.** Avoid "X is stable" — use "X is covered by test Y" or "smoke test Z passes."
- **§11 Implementation checkpoints maps work to the sdi-mode discipline.** Both the canonical gates (mirrored from sdi-mode for self-containment) and phase-specific slots appear inline. Only add phase-specific gates when the phase has constraints unique to it; don't restate the standard gates.

> **Maintenance note (for framework maintainers, not for inclusion in generated plans):** §11's standard gates are mirrored from the sdi-mode skill's `stop-and-review-patterns.md`. All three planning skills (`mvp-architect`, `convert-to-sdi`, `sdi-next-plan`) carry their own copy of this template on purpose so a generated plan can be reviewed without sdi-mode installed. When the canonical gates change, update all three copies (`mvp-architect/references/core-templates/implementation-plan-template.md`, `convert-to-sdi/references/core-templates/implementation-plan-template.md`, and `sdi-next-plan/references/core-templates/implementation-plan-template.md`) to match. Same convention as `agents-template.md`.
- **§12 Decisions Log is a checklist for end-of-phase housekeeping.** Each becomes a real DECISIONS.md entry at phase close.
- **§13 is where the plan is honest about its own limitations.** Usually empty at initial draft; populated during audit.

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
- **Empty §11 Implementation checkpoints.** A plan without checkpoint mapping forces the implementer to retrofit the discipline at execution time. Map every §2–§9 work unit to a checkpoint, drop checkpoints that don't apply, and always keep Checkpoints 1 and 5.
- **Missing the kickoff prompt.** §"How to feed this to the coding agent" is valuable — users copy-paste it. If the plan doesn't have one, the user has to invent it, and they'll miss things.

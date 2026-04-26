# Artifact Bundle

The standard set of artifacts produced at the end of Phase C. Each has a specific purpose — they don't duplicate each other. The bundle adapts based on the project type chosen in Phase 0.

## The artifacts

1. **README.md** — index. Links to the others. One-paragraph product summary. Stack at a glance. Project type + AI modifier flag. Open questions deferred to production.

2. **PRD.md** — what's being built and why. Users, roles, features in scope, features explicitly out of scope, success metrics. Universal across project types.

3. **ARCHITECTURE.md** — how the system is designed. Universal sections (stack, critical flows, security, observability, trade-offs) **plus** type-specific sections inserted from `project-types/{type}/architecture-appendix.md` (multi-tenancy + data model + RLS for SaaS, rendering strategy for landing pages, data sources + caching for dashboards, etc.).

4. **ROADMAP.md** — phased delivery plan. Each phase gets a goal, deliverables, and acceptance criterion ("done when..."). Includes a fast-track option if relevant. Universal across project types.

5. **PROJECT_STRUCTURE.md** — repo layout (directory tree with file roles), coding conventions, environment variables, kickoff prompt template at the bottom. Comes from `project-types/{type}/project-structure-template.md` (heavily type-specific).

6. **DESIGN_SYSTEM.md** — typography, color tokens, spacing, motion, component patterns, accessibility targets. Comes from `project-types/{type}/design-system-template.md`. **Only generated for project types with UI** (web-saas, landing-page, dashboard, mobile). Skip entirely for headless types (api-service, data-pipeline, automation-workflow, ai-agent without UI).

7. **IMPLEMENTATION_PLAN_PHASE_1.md** (or `IMPLEMENTATION_PLAN_<slug>.md`) — detailed implementation spec for the first work item. This is the artifact the coding agent consumes most directly. Much more prescriptive than the others. Universal core (`core-templates/implementation-plan-template.md`) merged with type-specific sections.

   **Naming:** the framework treats `IMPLEMENTATION_PLAN_*.md` uniformly. Use `PHASE_N` for projects with a discrete linear roadmap (greenfield, structured migrations) and `<slug>` for free-form work in ongoing projects (features, maintenance, perf passes). Phase E of this skill (and the `convert2sdi` skill for legacy projects) generate plans for subsequent work items, picking the appropriate naming.

   **ROADMAP coupling:** `ROADMAP.md` uses "Phase 0/1/2..." structure and is generated when a project has discrete planned phases. Continuous-feature or maintenance-mode projects may skip ROADMAP entirely or keep it minimal — work items are scoped one at a time via Phase E.

8. **AGENTS.md** *(at repo root, not in `docs/`)* — operating discipline for the coding agent during implementation. Generated from the template at `sdi-mode/adapters/claudecode-codex/AGENTS.md` and customized with the project's type, AI modifier flag, stack, and known conventions. For Claude Code / Codex this is the primary discipline anchor. For Roo Code / Kilo Code / OpenCode users, the equivalent is the `sdi-mode` custom mode configured per `sdi-mode/adapters/{tool}.md`; AGENTS.md still gets generated for documentation parity.

## Conditional generation

| Artifact | Always generate? | Notes |
|---|---|---|
| README, PRD, ARCHITECTURE, ROADMAP, PROJECT_STRUCTURE, IMPLEMENTATION_PLAN_PHASE_1, AGENTS.md | Yes | Universal |
| DESIGN_SYSTEM | Only if UI | Skip for api-service, data-pipeline, automation-workflow, ai-agent (without UI) |
| AI-modifier additions | Only if modifier active | Adds sections to ARCHITECTURE, PROJECT_STRUCTURE, DESIGN_SYSTEM, IMPLEMENTATION_PLAN_PHASE_1 |

## Order and dependencies

Generate in this order — each builds on the previous:

```
PRD → ARCHITECTURE → ROADMAP → PROJECT_STRUCTURE → DESIGN_SYSTEM (if UI) → IMPLEMENTATION_PLAN_PHASE_1 → AGENTS.md → README
```

README is last because it's an index — it can't index documents that don't exist yet. AGENTS.md is generated near-last because it benefits from knowing the stack/structure decisions from PROJECT_STRUCTURE and the Phase 1 prerequisites from IMPLEMENTATION_PLAN.

## Relationships

- **PRD** introduces the roles, features, and out-of-scope items. Everything else refers back.
- **ARCHITECTURE** operationalizes PRD features into stack and flows. Type appendix fills the structural model (multi-tenancy, data sources, ingestion pipeline, etc.).
- **ROADMAP** sequences the features from ARCHITECTURE into deliverable phases.
- **PROJECT_STRUCTURE** tells the coding agent where things live. Used by every subsequent plan.
- **DESIGN_SYSTEM** is consumed by UI-producing tasks; non-UI phases can ignore it.
- **IMPLEMENTATION_PLAN_PHASE_1** is the first "how" doc. Subsequent phases get their own plans later, not all at once.
- **AGENTS.md** sits at repo root and is the discipline carrier — it tells the coding agent how to operate, where to find docs, and what conventions to follow. It evolves as the project proceeds (the coding agent enriches it with discovered repo conventions).
- **README** indexes everything for human readers.

## What NOT to include in the bundle

- **All implementation plans at once.** Only Phase 1 is detailed now. Plans for later phases are speculative and will go stale. Generate them just before each phase starts.

- **Full test specifications.** The plan mentions required tests by category. Fleshed-out test cases emerge during implementation.

- **API documentation.** Generate from code later (Swagger/TypeDoc). At planning time, API contracts live in ARCHITECTURE and IMPLEMENTATION_PLAN.

- **User manual / end-user docs.** MVP ships without these; if the user wants them, that's a separate task.

- **Infrastructure runbooks, monitoring playbooks, incident response.** Phase 6-ish concerns.

- **DECISIONS.md upfront.** This document is populated by the coding agent during implementation, not preloaded. Mention it in PROJECT_STRUCTURE and AGENTS.md as a convention, don't generate it.

## Language

Default to **English** for artifacts even if the conversation is in another language. Reasons:

- Code, comments, and UI will be in English anyway (easier i18n later).
- Coding agents (Claude Code, Codex, Roo Code, etc.) are measurably stronger on English-aligned docs.
- Future collaborators may not share the user's language.

Say so when announcing artifacts: "Generating artifacts in English because [reasons]. If you prefer another language, say so."

## Scope of applicability check

Before generating, confirm the project type chosen in Phase 0 still matches what was discovered in Phase A/B. If the user pivoted (e.g., started as "landing page" and grew into "web SaaS multi-tenant"), say so explicitly and re-route to the appropriate type playbook before generating.

## Length discipline

The artifacts are decision records, not encyclopedias. Rules of thumb:

- **PRD**: ~400–600 lines. Each section is crisp.
- **ARCHITECTURE**: ~500–700 lines including type appendix. Flows are walkthrough-style, not pseudocode.
- **ROADMAP**: ~200–300 lines. Phases are tight.
- **PROJECT_STRUCTURE**: ~250–450 lines (varies by type — landing pages tighter, web SaaS larger).
- **DESIGN_SYSTEM**: ~250–400 lines.
- **IMPLEMENTATION_PLAN_PHASE_1**: ~400–600 lines. This one is the most prescriptive; being longer is OK.
- **AGENTS.md**: ~150–250 lines. Operating discipline + project-specific conventions.
- **README**: ~30–80 lines. Strictly an index.

If you're trending past these, ask yourself what can be cut. Over-long docs are ignored.

## Announcing completion

After the last artifact is generated, give the user:

1. **One-paragraph recap** of what was generated and how to read it.
2. **How to use with the coding agent** — point to the kickoff prompt template, with both Path A (AGENTS.md for Claude Code/Codex) and Path B (custom mode for Roo/Kilo/OpenCode) explained.
3. **What's next** — typically Phase 0 scaffolding or, for projects with significant UI, a UI prototype (Claude Design or equivalent) before code.

Keep the follow-up short. The artifacts are the deliverable.

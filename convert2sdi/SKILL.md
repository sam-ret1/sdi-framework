---
name: convert2sdi
description: Adopt the sdi-framework workflow on a project that already has code — without refounding it. Auto-audits the repo, asks ≤5 focused questions (each accepts "don't know"), then generates the artifact bundle: AGENTS.md, PROJECT_STRUCTURE, ARCHITECTURE, DECISIONS seed, MEMORY init, PRD/ROADMAP placeholders, first IMPLEMENTATION_PLAN. For projects in production, emits a `Production constraints` section in AGENTS.md so sdi-mode behaves safely against live systems. Use when the user says "I have a project already in progress and want to use the framework", "taking over this legacy repo", "need to create AGENTS.md here", or any variant where the project has existing code and the user wants to adopt the SDI discipline going forward. Entry skill for ANY project not started via `mvp-architect` Phase 0–C.
---

# convert2sdi

This skill is how an existing project adopts the sdi-framework workflow. It does not refactor the code, rewrite history, or interrogate the user about decisions made years ago. It documents the reality of the repo as it stands today, accepts gaps honestly, and hands the project over to `sdi-mode` so all future work follows the discipline.

## When to use

- The project has existing code (any size — from a 2-week MVP to a 5-year SaaS in production).
- The user wants to adopt the SDI workflow going forward but didn't start with `mvp-architect` Phase 0–C.
- The user is taking over an inherited project and wants to standardize how it'll be worked on.

**Do NOT use** for:
- Greenfield projects with no code yet — use `mvp-architect` (Phase 0–C).
- Reviewing or refactoring existing code — convert2sdi only generates docs; it never edits source code.
- Single-file scripts or throwaway prototypes — overhead exceeds value.

## Core principle

**Document reality. Never invent rationale. Accept thin artifacts. Move forward.**

This skill exists because the user's primary need is *operational* — get the project under the framework so future work is disciplined. It is **not** archeological — there's no obligation to recover historical reasoning that wasn't written down.

When information is missing:
1. Try to detect from code automatically (best effort).
2. If detection fails, ask the user — but accept "don't know / skip" without follow-up.
3. If both fail, mark the artifact section with a confidence flag and move on.

The goal is a working framework setup in <30 minutes of user time, not a perfect retrospective.

## The flow — Phase 0/1/2/3 (+ Phase 1.5 when applicable)

### Phase 0 — Auto-audit (no user input)

The skill scans the repo programmatically and produces a **discovery report**. No questions yet.

Read `references/auto-audit-checklist.md` for the full checklist. Categories detected:

- **Project type** (mapped to one of the framework's 8 types)
- **Stack** (language, framework, ORM, database, auth, deployment)
- **Conventions** (file layout, test runner, linting, commit style)
- **Existing canonical artifacts + format compatibility** (`README`, `PRD`, `ARCHITECTURE`, `ROADMAP`, `PROJECT_STRUCTURE`, `DESIGN_SYSTEM`, `DECISIONS`, `AGENTS`, ADR folders) — classified as compatible / partially-compatible / incompatible
- **Other docs** (CHANGELOG, custom `docs/*.md`, IDE rule files) — inventoried; preserved unconditionally
- **Production indicators** (semver tags, CHANGELOG with multiple versions, monitoring config, feature flag systems, billing/multi-tenancy in code)
- **AI modifier signals** (LLM SDKs, prompts/, evals/)

Output: discovery report shown to user as a structured summary, with confidence per finding. The user reads it and either confirms or corrects in Phase 1.

### Phase 1 — Quick triage (≤5 questions)

Read `references/triage-questions.md`. Ask the 5 questions in one batch (or one at a time if the user prefers). **Every question accepts "don't know / skip" with a defined skip behavior.**

The 5 questions cover:

1. Project type confirmation (vs auto-detected)
2. AI modifier active?
3. Project stage (greenfield-mvp / mvp-launched / mature-production / maintenance / unknown)
4. Work cadence (discrete phases / continuous features / mostly maintenance / mixed / unknown)
5. Anything material the audit might have missed (clients, contracts, compliance, deadlines, on-call)

Do **not** ask additional questions proactively. If the user volunteers more context, capture it; otherwise move on.

### Phase 1.5 — Existing artifact triage (only if Phase 0 detected pre-existing canonical artifacts)

This phase **runs only when needed**. Skip entirely if Phase 0 found no existing canonical artifacts (or all detected ones are absent / clearly compatible with the framework). When it does run, it prevents `convert2sdi` from silently overwriting the user's PRD, ARCHITECTURE, ROADMAP, etc.

Read `references/existing-artifact-handling.md` for the full protocol, including the matrix of default strategies, format compatibility heuristics, and the four canonical handling strategies:

- **Strategy A — Preserve as-is + framework header.** Existing file already does its job; framework just adds a marker.
- **Strategy B — Convert in place.** Merge canonical structure with existing content; show diff; user approves.
- **Strategy C — Rename existing + generate new.** Rename existing to `<name>.legacy.md`, generate fresh canonical version, link the legacy.
- **Strategy D — Skip generation entirely.** Don't generate the artifact; document that source of truth lives elsewhere (Notion, Linear, Figma, etc.) in AGENTS.md "External artifact references".

The flow:

1. **Present the matrix** — show the user a table with each detected canonical artifact, its compatibility class, and the proposed default strategy. Include ADR folders (special case — default to Option 1: keep ADRs + DECISIONS.md as index) and any custom `docs/` content (default to Strategy A).

2. **User responds** — accepts defaults ("ok", "go") or overrides per-file ("ROADMAP: keep as-is", "PRD: skip — lives in Notion at [link]").

3. **Confirm and proceed** — echo back the final per-file plan, then enter Phase 2 with strategies applied.

The skill **never silently overwrites** a canonical artifact. Even if the user says "go" without reading the matrix, every Strategy B (convert in place) requires showing the diff during Phase 2 generation before writing.

If everything in the matrix maps to Strategy A (all compatible) and there's no ADR / external-tool integration to confirm, this phase is brief — basically a "I detected these existing files; they're all compatible; will preserve + add framework header. ok to proceed?" message.

### Phase 2 — Generate artifacts

Read `references/artifact-generation-rules.md` and `references/confidence-flags.md`. Generate the bundle:

- **AGENTS.md** (rich) — stack, conventions, document map, work tracker (empty), `Production constraints` section if stage = production (template at `references/production-constraints-template.md`).
- **PROJECT_STRUCTURE.md** (rich) — documents the repo as it actually is.
- **ARCHITECTURE.md** (mixed) — §Stack and §Critical flows extracted from code; §Type-specific appendix populated as far as code reveals; §Trade-offs as placeholder.
- **DESIGN_SYSTEM.md** (medium, only if UI) — extracted tokens and components; aesthetic intent left as placeholder.
- **DECISIONS.md** (seed, 3–5 entries) — obvious patterns detected (e.g. "uses Drizzle ORM", "monorepo with pnpm workspaces"). Marked `Source: code analysis — confirm rationale with team`.
- **docs/MEMORY.md + docs/memory/YYYY-MM-DD.md** (fresh) — index empty except for today's entry, which describes the framework setup itself.
- **PRD.md** (thin by design) — `Current state` (extracted from code/README) + `Out of scope` placeholder.
- **ROADMAP.md** (optional) — only if Q4 = "discrete phases planned" and the user has visibility into future work; otherwise minimal note pointing to Phase E of `mvp-architect` for forward planning.

Each artifact carries a confidence flag at the top indicating the source (code analysis / user-confirmed / best-effort placeholder).

### Phase 3 — Plan first work item → handoff to sdi-mode

Read `references/first-work-item.md`. Ask the user what the next concrete work is:

- Feature
- Bugfix / hotfix
- Migration / structured refactor
- Performance / observability pass
- Other

Generate the corresponding `IMPLEMENTATION_PLAN_*.md`:

- Migration / structured refactor with linear plan → `IMPLEMENTATION_PLAN_PHASE_N.md` (where N is the next number, often 1 or 2 if the project hasn't tracked phases before).
- Feature / bugfix / perf / other → `IMPLEMENTATION_PLAN_<slug>.md`.

The plan's §0 Pre-requisites lists the **current production state as the foundation** — not "Phase N-1 delivered". The plan is grounded in the live repo.

After generation, instruct the user how to load `sdi-mode`:
- Claude Code / Codex: AGENTS.md is already at repo root; paste the kickoff prompt.
- Roo Code / Kilo Code / OpenCode: configure the custom mode per `sdi-framework/sdi-mode/adapters/{tool}.md`, then paste the kickoff prompt.

Closing message:

> "Setup complete. The project is now under the framework. For the next work item after this plan closes, use Phase E of the `mvp-architect` skill (don't run `convert2sdi` again — it's one-shot)."

## Tone

- **Matter-of-fact, no interrogation.** Each question is one ask, accepting "don't know" gracefully.
- **No apologetics for thin artifacts.** Mark them clearly and move on. The framework is forward-looking.
- **No surprise edits to code.** This skill never modifies source files. It only generates docs.
- **Conversational language.** Reply in the user's conversational language. The English message templates in this skill and its references are authoring shells — render them in the user's language when speaking to them. Generated artifacts still default to English (per the framework convention) unless the user asks otherwise.

## What this skill is NOT

- **Not a refactor agent.** Doesn't change code structure, naming, or organization.
- **Not a documentation rewrite.** Existing docs (README, ARCHITECTURE, etc.) are preserved if they exist; the skill complements them.
- **Not a scope-from-scratch tool.** Doesn't replace `mvp-architect` Phase 0–C. If the user wants to re-scope the product, that's a separate exercise.
- **Not an audit tool for code quality.** SDI mode does its own audit per work item; convert2sdi only sets up the framework.
- **Not run repeatedly.** One-shot per project. Subsequent work uses Phase E of `mvp-architect`.

## Smoke check after run

User should be able to:

1. Open `AGENTS.md` and recognize their project (stack, type, conventions).
2. Open the generated `IMPLEMENTATION_PLAN_*.md` and see a plan grounded in current code.
3. Start a session in their coding agent (Claude Code / Codex / Roo / Kilo / OpenCode), paste the kickoff prompt, and have the agent operate with SDI discipline.

If any of those fail, the convert2sdi run is incomplete — investigate before considering it done.

## References

Load these as needed:

- `references/auto-audit-checklist.md` — Phase 0: what to detect automatically (including format compatibility for existing canonical artifacts)
- `references/triage-questions.md` — Phase 1: the 5 questions with skip behavior
- `references/existing-artifact-handling.md` — Phase 1.5: matrix + 4 strategies (A/B/C/D) for existing PRD/ARCHITECTURE/ROADMAP/etc. + ADR special handling
- `references/confidence-flags.md` — how to mark thin sections in artifacts
- `references/artifact-generation-rules.md` — Phase 2: per-artifact rules and confidence levels
- `references/production-constraints-template.md` — section emitted in AGENTS.md when stage = production
- `references/first-work-item.md` — Phase 3: handoff to sdi-mode

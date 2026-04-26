# Kickoff Prompt Template

At the end of Phase C, the user takes the spec bundle and hands it to a coding agent. The kickoff sets the tone for how the agent operates. There are **two paths** depending on the tool — pick the one matching the user's setup, or generate both if unclear.

## Path A — Claude Code / Codex

Used when the project has `AGENTS.md` (and `CLAUDE.md` if Claude Code) at the repo root, generated from the bundle. The discipline (audit-first, stop-and-review, DECISIONS.md, round reports) loads automatically because the agent reads `AGENTS.md` (Codex) or `CLAUDE.md` (Claude Code, which forwards to AGENTS.md) on session start.

Paste at the start of the user's first session:

```
Implementing [Phase 1 / <slug>] of this project per docs/IMPLEMENTATION_PLAN_[PHASE_1 | <slug>].md.

Read in this order before writing any code:
1. README.md to get context.
2. AGENTS.md (already loaded as context, but verify the conventions).
3. docs/IMPLEMENTATION_PLAN_[PHASE_1 | <slug>].md fully.
4. docs/PROJECT_STRUCTURE.md for conventions.
5. docs/ARCHITECTURE.md — at minimum the type-specific section and the critical flows.

Then audit the plan against the actual repo state. Produce an audit report covering:
- Blockers (things preventing start)
- Plan-repo divergences (where plan and repo disagree — the repo wins)
- Open questions for me
- Aligned / no action items

After the audit, propose the first deliverable — usually [foundation: schema/migration/dependencies for types with a database; project scaffolding for fresh repos] — and stop for review. Don't start on subsequent scope items until I approve the foundation.
```

This prompt is short because `AGENTS.md` already carries the SDI discipline (the 8 steps, the not-rules, the tone). The kickoff just signals "we're starting this work item — proceed with audit."

Substitute `[Phase 1 / <slug>]` with whichever naming applies — `PHASE_N` for discrete phases (greenfield/migrations) or `<slug>` for free-form work (features/maintenance in ongoing projects).

## Path B — Roo Code / Kilo Code / OpenCode

Used when the project relies on a configured `sdi-mode` custom mode rather than `AGENTS.md`. The custom mode is set up beforehand following `sdi-mode/adapters/{tool}.md`.

Paste at the start of the user's first session, with `sdi-mode` active:

```
Implementing [Phase 1 / <slug>] of this project per docs/IMPLEMENTATION_PLAN_[PHASE_1 | <slug>].md.

Read in this order before writing any code:
1. README.md to get context.
2. docs/IMPLEMENTATION_PLAN_[PHASE_1 | <slug>].md fully.
3. docs/PROJECT_STRUCTURE.md for conventions.
4. docs/ARCHITECTURE.md — at minimum the type-specific section and the critical flows.

You are operating under SDI mode discipline (loaded as your system prompt). Audit the plan against the actual repo state and produce an audit report. After the audit, propose the first deliverable and stop for review.
```

This prompt is also short because the `sdi-mode` system prompt (from `MODE.md`) already carries the discipline.

## Variations for mid-project prompts

### Starting Phase N (where N > 1)

```
Phase N-1 is shipped (see DECISIONS.md and PROJECT_STRUCTURE.md for what was delivered).

Now implementing Phase N per docs/IMPLEMENTATION_PLAN_PHASE_N.md.

Follow the same SDI discipline as Phase 1:
1. Read the plan fully.
2. Audit against the current repo.
3. Produce an audit report before writing code.
4. Stop at natural checkpoints for review.
5. Write tests alongside code.
6. DECISIONS.md for non-obvious choices.

Start with the audit.
```

### Continuing within a work item (phase or slug)

```
Continuing [Phase N / <slug>] from where we stopped. Last round delivered [X]; next suggested round is [Y].

Before starting, verify:
- Round [X] is fully merged and tests are green.
- Audit state from the start of the phase is still valid (no new divergences).
- Plan §[section] for round [Y] has what you need, or needs a revision note.

Propose the first deliverable of this round and stop for review.
```

## How to adapt

- If the user uses a coding agent not listed here (Cursor, Continue, Cody, Aider, etc.), adapt phrasing but keep the ground rules intact: read first, audit before code, stop at checkpoints, DECISIONS.md, repo wins. If the tool supports a `.cursorrules`/`.continuerc`/etc. file, consider porting AGENTS.md content to that format.
- If the project doesn't have a UI, skip the DESIGN_SYSTEM reference.
- If the user wants to start from a phase other than Phase 1 (e.g. continuing an existing project), use the "Starting Phase N" variation.

## What the prompts do

- **Force context loading before code.** The agent reads before writing.
- **Prime the audit discipline.** The audit is the highest-leverage step; make it explicit.
- **Establish repo-wins rule.** Prevents the agent from being hostage to a stale plan.
- **Pre-announce stop-and-review.** The user isn't surprised when the agent stops; it's expected.
- **Normalize DECISIONS.md.** Makes the paper trail a default, not an afterthought.
- **Set tone for revision notes.** Plans evolve; the agent knows how to mark that.

The two paths differ only in *how* the discipline gets loaded (AGENTS.md vs. mode system prompt). The behaviors expected from the agent are identical.

## When NOT to use the kickoff prompt

- If the project is a simple one-off task (script, single component, throwaway prototype), this is overkill.
- If the user is pair-coding interactively and doesn't want the audit-first discipline (short tasks, exploratory work), respect that — neither the skill nor the mode is meant to be coercive.

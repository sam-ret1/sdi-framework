# Expected Artifacts

What the spec bundle should contain when you receive it. If it's incomplete, flag that before starting.

## The bundle

At minimum, you should find under `docs/` (or equivalent):

- `README.md` — index, linking to the others.
- `PRD.md` — product requirements.
- `ARCHITECTURE.md` — stack, type-specific structural model, flows, trade-offs.
- `ROADMAP.md` — phases with goals and acceptance criteria.
- `PROJECT_STRUCTURE.md` — repo layout and conventions.
- `IMPLEMENTATION_PLAN_*.md` — detailed spec for the current work item. The framework treats `IMPLEMENTATION_PLAN_*.md` uniformly: `PHASE_N` for discrete phases (greenfield, structured migrations) or `<slug>` for free-form work (features, maintenance batches in ongoing projects).

At repo root:

- `AGENTS.md` — operating discipline + project-specific stack/conventions. Required when running via Claude Code or Codex; optional but recommended for parity when running via Roo Code/Kilo Code/OpenCode (which use mode metadata as the discipline carrier).
- `CLAUDE.md` — present when using Claude Code; should just point to AGENTS.md.

Optionally:

- `DESIGN_SYSTEM.md` — only if the project has a UI.
- `DECISIONS.md` — atemporal paper trail. May or may not be populated at Phase 1 start; you'll be writing to it throughout.
- `MEMORY.md` + `memory/YYYY-MM-DD.md` — datable session memory. May not exist at Phase 1 start; you create on first working session. See `memory-discipline.md`.

## Mode discipline carrier

Different tools carry the SDI discipline differently:

| Tool | Discipline carrier |
|---|---|
| Claude Code | `CLAUDE.md` → `AGENTS.md` (full SDI rules embedded) |
| Codex | `AGENTS.md` (read natively) |
| Roo Code / Kilo Code / OpenCode | Custom mode `sdi-mode` configured per `sdi-mode/adapters/{tool}.md` |

If neither AGENTS.md nor a configured custom mode is present, the SDI discipline isn't loaded. Ask the user before proceeding.

## How to recognize a complete handoff

Before starting a phase, verify:

1. **An `IMPLEMENTATION_PLAN_*.md` exists** for the work item the user is asking about (e.g. `IMPLEMENTATION_PLAN_PHASE_2.md` if they say "implement phase 2", or `IMPLEMENTATION_PLAN_billing-portal.md` if they say "implement the billing portal feature"). If no matching plan exists, stop and ask — don't infer one from the roadmap or prose alone.

2. **The plan has concrete sections**, not placeholders. Look for: schema/structural sketches, contracts with status codes (for APIs), test requirements, acceptance criteria. If most sections are TODO, the handoff is incomplete.

3. **PROJECT_STRUCTURE.md matches the actual repo.** Drift here indicates the docs weren't updated after recent work — audit will surface this anyway.

4. **PRD.md out-of-scope section is explicit.** If the PRD doesn't say what's *not* in the MVP, you'll likely over-build.

5. **AGENTS.md (or active mode) exists** so the discipline is loaded. If not, surface this to the user before coding.

## What to do when artifacts are missing or thin

### Missing IMPLEMENTATION_PLAN for the work item

Don't improvise. Stop and tell the user:

> "There's no `IMPLEMENTATION_PLAN_*.md` for [phase / work item]. I can either: (a) draft one based on the ROADMAP / current repo state and ask you to review before starting; or (b) you can generate one via the `mvp-architect` skill — Phase E for next-phase planning in an ongoing project, or full Phase 0-C for a new direction. Which do you prefer?"

Option (a) is OK for small, well-contained work items. Option (b) is better when the work introduces new architectural ideas or deserves the full discovery flow.

### Plan has gaps

If the plan has sections but some are thin or ambiguous, list the gaps as **open questions** in the audit report. Don't silently fill them in.

### PROJECT_STRUCTURE.md is outdated

Flag it. Offer to update it as part of the phase's housekeeping round. Don't let "docs are wrong" become "docs stay wrong" — this is how spec-driven projects lose coherence.

### DECISIONS.md doesn't exist

Create it at the start of the phase. The first entry is your revision note or your first audit finding.

### AGENTS.md missing or sparse

If the file exists but only has the bare template (placeholders unfilled), expect to fill in stack and conventions during the audit. If the file is missing entirely on a Claude Code/Codex setup, surface it: "There's no AGENTS.md at the repo root. Generate one from `mvp-architect`'s template before I proceed, or I'll work without project-specific discipline anchored at the repo level."

## Reading order for a new phase

1. **AGENTS.md** (if present) — 2 minutes. Stack and project-specific conventions.
2. **MEMORY.md** (if present) + last 2–3 daily entries from `memory/` — 3 minutes. Tells you where the work actually is right now, what's blocked, what's pending. Faster than re-reading the plan to figure out current state.
3. **README.md** — 2 minutes. Gives you product and stack context.
4. **The current `IMPLEMENTATION_PLAN_*.md`** (`PHASE_N` or `<slug>`) — top to bottom. This is your primary spec.
5. **Relevant sections of ARCHITECTURE.md** — type-specific section + critical flows for this phase.
6. **PROJECT_STRUCTURE.md** — skim for conventions; bookmark the coding conventions section.
7. **DECISIONS.md** — skim headers; read any decisions that apply to the phase.
8. **Relevant repo files** — the schemas, helpers, and modules the phase will touch.

This is ~15–30 minutes of reading before you start coding. Don't skip it.

## What the planner artifacts mean (interpretation guide)

Because the planner skill (`mvp-architect`) and this implementation mode are designed together, some artifact sections have specific meaning:

- **`§ Decisions Log (for DECISIONS.md)` in the plan** — these are decisions *the plan anticipates making or memorializing* once implementation is done. Treat them as a checklist; each becomes a real DECISIONS.md entry at end of phase (or earlier if clarified).

- **`§ Known divergences` in the plan** — places where the plan itself acknowledges it disagrees with the repo or with other docs. At end of phase, mark each as ✓ resolved or ⏸ intentionally deferred.

- **"Revision note (r2, r3...)"** — the plan has been updated during implementation. The note summarizes what changed. Always read these; they're context you'd otherwise miss.

- **"Out of scope" in PRD** — these are not features to "sneak in because it's easy." If a user asks for something in that list, refer to the list and ask whether to escalate to a scope change or defer.

- **"Not done in this round (and why)" in a round report** — the previous round explicitly deferred these items. They are real work items, just not in the scope of that round. Carry them forward.

## Document precedence

Docs disagree. When two say different things, you need a rule for which wins. Without one, the agent silently picks the wrong source — and the user can't tell that happened.

The hierarchy below is from highest authority (top) to lowest (bottom). When two docs conflict, the higher one wins. The lower one gets a revision note (or end-of-phase update) to align.

| Priority | Source | Authority |
|---|---|---|
| 1 | **Live repo state** (committed code) | Wins over every doc. Reality is the canonical truth. |
| 2 | **AGENTS.md** / mode metadata | Current operating discipline + project-specific conventions. Wins over the planning docs because it's continuously updated to reflect actual repo state. |
| 3 | **PRD.md** | What & why. Establishes scope and intent. Higher than how/when because changing it implies a re-scope. |
| 4 | **ARCHITECTURE.md** | How — stack, type-specific structural model, critical flows. Higher than the rest because it's the technical contract. |
| 5 | **ROADMAP.md** | When — phases and acceptance criteria. |
| 6 | **PROJECT_STRUCTURE.md** | Where — file conventions, repo layout. |
| 7 | **IMPLEMENTATION_PLAN_PHASE_N.md** | Detailed how, phase-scoped. Lower than the higher docs because it's a tactical translation; if it disagrees with ARCHITECTURE, the plan is wrong, not the architecture. |
| 8 | **DESIGN_SYSTEM.md** | Visual language (UI types only). Lower than the higher docs because design must serve product, not vice versa. |
| 9 | **README.md** | Index. Never source of truth — if README disagrees with anything, README is wrong. |
| – | **DECISIONS.md** | *Patches and exceptions*, not authority. See note below. |
| – | **memory/YYYY-MM-DD.md** | *Breadcrumb trail*, not authority. See note below. |

### DECISIONS.md is overlay, not override

A `DECISIONS.md` entry doesn't override a higher-level doc — it documents an exception or a concretization. If a decision contradicts the higher-level doc persistently, the higher doc must be updated:

- **Pattern A — local exception** (decision compatible with the doc): "DECISIONS #18 — RLS bypass intentional for this code path because ..." This is a local exception logged for traceability. ARCHITECTURE.md still says "RLS is the primary defense"; the decision documents the exception.
- **Pattern B — doc needs update** (decision contradicts the doc): "DECISIONS #42 — switching from Postgres to MongoDB." This is not a local exception. It's a contradiction; ARCHITECTURE.md is now wrong. Either revert the decision or update ARCHITECTURE.md to align (with a revision note in the plan if mid-phase).

Don't hide contradictions in DECISIONS.md. If you find one, surface it.

### memory/YYYY-MM-DD.md is breadcrumb, not authority

Memory entries describe *what was happening on a given day*. They never override a doc. If memory says "Round B done" and PROJECT_STRUCTURE doesn't reflect Round B's new directory, PROJECT_STRUCTURE needs update — not memory.

### How to apply the rule

When you find a conflict during the audit (or mid-phase):

1. **Identify the higher-priority doc** per the table above.
2. **The higher wins.** Use its statement as the truth for code and decisions.
3. **The lower doc must be aligned.** Either:
   - Add a revision note to the lower doc (`r2`, `r3`...) summarizing the change.
   - Schedule the alignment for end-of-phase housekeeping if not blocking.
   - If the lower doc is the plan, the audit step already handles this.
4. **Add a `DECISIONS.md` entry** if the conflict resolution required a non-obvious choice.

### Special cases

- **Live repo vs ARCHITECTURE.md disagree:** repo wins by definition (priority 1). But ask: did the repo drift accidentally, or was a deliberate decision made that wasn't documented? If accidental drift, ARCHITECTURE wins and you fix the repo. If deliberate, ARCHITECTURE needs update + a `DECISIONS.md` entry.
- **AGENTS.md vs PROJECT_STRUCTURE.md disagree:** AGENTS.md wins on stack/conventions (it's the live operating manual). But propose updating PROJECT_STRUCTURE during housekeeping so external readers don't get confused.
- **PRD vs IMPLEMENTATION_PLAN disagree on a feature:** PRD wins. The plan is a translation, not an authority. Either the plan needs revision, or the PRD needs an explicit out-of-scope clause to deprecate the feature — not silent drift.
- **Two same-level docs disagree** (e.g. ARCHITECTURE vs PROJECT_STRUCTURE on file paths): rare, but it happens. The one tied more directly to running code wins (PROJECT_STRUCTURE in this case). Update the other.

### What this rule prevents

- Silent drift where the agent picks "whichever doc was easier to find."
- Plan-vs-architecture contradictions that compound across phases.
- DECISIONS.md becoming a graveyard of contradictions to higher-level docs that no one ever resolves.

## Talking to the planner agent

The user may pair you with a second agent acting as the `mvp-architect` skill, consultatively during implementation. If the user relays a question or takes audit suggestions back to the planner, that's normal. Don't get defensive; it's a good-faith review loop. Respond to the planner's feedback the same way you respond to the user's.

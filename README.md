# SDI Framework

> **Spec-Driven Implementation**: a workflow for using coding agents on software projects, from rough idea to shipped feature.

SDI is opinionated about *how* work is done, not *what* the product becomes. It combines two planning skills with one execution discipline so a vague idea, or an existing codebase, can become a clear, auditable, reviewable engineering workflow.

It is stack-agnostic, single-developer friendly, and designed for IDE-based AI coding tools.

## The Pieces

| Piece | Type | Purpose | Use when |
| --- | --- | --- | --- |
| `mvp-architect` | Skill | Turn an idea into a spec bundle; review work in progress; plan the next work item. | Starting greenfield, reviewing implementation, or planning the next phase/feature. |
| `convert-to-sdi` | Skill | Adopt SDI on a project that already has code. | Joining a legacy project or formalizing an in-flight project. |
| `sdi-mode` | Execution discipline | Implement against specs with audit-first checkpoints, decisions, memory, and document precedence. | Writing code from an SDI plan. |

The skills handle planning. The mode handles implementation.

## Quick Start

Choose the path that matches your project:

| Situation | Use |
| --- | --- |
| Starting from a rough idea | `mvp-architect` Phase 0 -> A -> B -> C, then load `sdi-mode`. |
| Adopting an existing codebase | `convert-to-sdi` Phase 0 -> 1 -> 1.5 if needed -> 2 -> 3, then load `sdi-mode`. |
| Implementing a planned work item | `sdi-mode` via `AGENTS.md`, custom mode, agent, or tool rule. |
| Reviewing work mid-implementation | `mvp-architect` Phase D. |
| Planning the next phase or feature | `mvp-architect` Phase E. |

After `convert-to-sdi` runs once on a project, do not run it again on that same project. Use `mvp-architect` Phase E for future plans.

## End-to-end Flow

### Greenfield

1. Start with a rough idea.
2. Run `mvp-architect` Phase 0 to pick the project type and optional AI/LLM modifier.
3. Run Phases A and B for discovery and trade-off resolution.
4. Run Phase C to generate the bundle: `PRD`, `ARCHITECTURE`, `ROADMAP`, `PROJECT_STRUCTURE`, optional `DESIGN_SYSTEM`, `IMPLEMENTATION_PLAN_PHASE_1`, `AGENTS.md`, and `README`.
5. Load `sdi-mode` and implement Phase 1 in audited rounds.
6. At the end of the phase, use Phase D for review if needed and Phase E to plan the next work item.

### Existing Codebase

1. Run `convert-to-sdi` Phase 0 for an automatic repo audit.
2. Answer Phase 1 triage questions, with "don't know" always accepted.
3. If existing docs are detected, use Phase 1.5 to decide how to preserve, convert, rename, or skip each artifact.
4. Run Phase 2 to generate the SDI bundle from the repo as it exists.
5. Run Phase 3 to create the first `IMPLEMENTATION_PLAN_*`.
6. Load `sdi-mode` and continue future work through the Phase D / Phase E loop.

## Installation

SDI has two install surfaces:

- **Planning skills**: `mvp-architect/` and `convert-to-sdi/`.
- **Execution discipline**: `sdi-mode/MODE.md`, adapted into the format your tool loads.

Use your tool's native skill, mode, agent, or rule system. The framework should live inside the tool you actually use; it should not require running planning in a separate tool and copying results back.

### Claude Code

Install the skills by copying `mvp-architect/` and `convert-to-sdi/` into one of:

- Project scope: `.claude/skills/mvp-architect/` and `.claude/skills/convert-to-sdi/`
- User scope: `~/.claude/skills/mvp-architect/` and `~/.claude/skills/convert-to-sdi/`

For execution, copy `sdi-mode/adapters/claudecode-codex/AGENTS.md` and `sdi-mode/adapters/claudecode-codex/CLAUDE.md` to the root of the target project. `CLAUDE.md` imports `AGENTS.md`.

### Codex

Install the skills by copying `mvp-architect/` and `convert-to-sdi/` into one of:

- Project scope: `.codex/skills/mvp-architect/` and `.codex/skills/convert-to-sdi/`
- User scope: `$CODEX_HOME/skills/mvp-architect/` and `$CODEX_HOME/skills/convert-to-sdi/` (defaults to `~/.codex/skills/...`)

For execution, copy `sdi-mode/adapters/claudecode-codex/AGENTS.md` to the root of the target project. Codex reads `AGENTS.md` natively. Restart Codex after adding skills so discovery refreshes.

### Roo Code, Kilo Code, and OpenCode

Configure `sdi-mode` through the tool's native custom-mode or agent system:

- Roo Code: [`sdi-mode/adapters/roocode.md`](sdi-mode/adapters/roocode.md) for `.roomodes`
- Kilo Code: [`sdi-mode/adapters/kilocode.md`](sdi-mode/adapters/kilocode.md) for `.kilo/agents/sdi.md`
- OpenCode: [`sdi-mode/adapters/opencode.md`](sdi-mode/adapters/opencode.md) for `.opencode/agents/sdi.md` or `opencode.json`

Roo Code's docs announce a product shutdown on May 15, 2026, so treat that adapter as support for existing Roo users rather than the default for new long-lived projects.

Use the same native mechanism for the planning skills when your tool supports reusable skills, agents, prompts, or commands. Add `mvp-architect/SKILL.md` and `convert-to-sdi/SKILL.md` as tool-specific skills, or adapt their contents into the closest equivalent. Keep the names `mvp-architect` and `convert-to-sdi` so project instructions and docs stay portable.

### Cursor, Cline, and Windsurf

Configure `sdi-mode` as an always-applied project rule:

- Cursor: [`sdi-mode/adapters/cursor.md`](sdi-mode/adapters/cursor.md) for `.cursor/rules/sdi-mode.mdc`
- Cline: [`sdi-mode/adapters/cline.md`](sdi-mode/adapters/cline.md) for `.clinerules/sdi-mode.md`
- Windsurf: [`sdi-mode/adapters/windsurf.md`](sdi-mode/adapters/windsurf.md) for `.windsurf/rules/sdi-mode.md`

For planning, install or adapt `mvp-architect/SKILL.md` and `convert-to-sdi/SKILL.md` through the tool's native skill, prompt, command, or rule mechanism. If a tool cannot load reusable planning instructions at all, it is not a supported planning target for this framework.

### Other Tools

Most coding agents can read either `AGENTS.md` or a tool-specific rules file. Use `sdi-mode/MODE.md` and `sdi-mode/adapters/claudecode-codex/AGENTS.md` as the source material, then adapt to the tool's expected format.

Quick examples:

- Continue.dev: use `.continue/rules/sdi-mode.md` or reference it from `.continue/config.yaml`.
- GitHub Copilot Chat: use `.github/copilot-instructions.md`.
- Aider: use `CONVENTIONS.md` and reference it from `.aider.conf.yml`.

Only document tools here when SDI can run inside that tool's own instruction system.

## Project Types

`mvp-architect` Phase 0 routes to one of eight playbooks:

1. Web app SaaS multi-tenant
2. Landing page / marketing site
3. Dashboard / admin / internal tool
4. Web API / backend service
5. Mobile app
6. Data pipeline / scraper + reports
7. AI agent / MCP server / chatbot
8. Integration / automation workflow

An optional AI/LLM modifier adds prompts, evals, cost controls, RAG, and guardrail concerns to any project type with significant AI behavior.

If the project does not fit any type, the skill asks whether to adapt on the fly or stop.

## Core Disciplines

These rules live in `sdi-mode/MODE.md` and propagate to projects through `AGENTS.md`, custom modes, agents, or rules.

1. **Audit the plan against the repo before coding.** Plans contain assumptions; the repo is reality. When they disagree, the repo wins and the plan gets a revision note.
2. **Stop at explicit checkpoints with binary gates.** Each phase has natural checkpoints, and each gate must pass before the round closes.
3. **Maintain decisions and memory separately.** Put durable rationale in `docs/DECISIONS.md`. Put dated work state, blockers, and next steps in `docs/memory/YYYY-MM-DD.md`.
4. **Respect document precedence.** Live repo state > `AGENTS.md` > `PRD` > `ARCHITECTURE` > `ROADMAP` > `PROJECT_STRUCTURE` > `IMPLEMENTATION_PLAN` > `DESIGN_SYSTEM` > `README`. `DECISIONS.md` patches authority; `docs/memory/` is a breadcrumb trail, not source of truth.

## Generated Artifacts

After `mvp-architect` or `convert-to-sdi`, a project usually has:

- `AGENTS.md`: operating discipline, stack, conventions, and project-specific constraints.
- `docs/PRD.md`: product scope and user-facing requirements.
- `docs/ARCHITECTURE.md`: system shape, critical flows, and trade-offs.
- `docs/ROADMAP.md`: phase or work-item progression.
- `docs/PROJECT_STRUCTURE.md`: repo layout and conventions.
- `docs/DESIGN_SYSTEM.md`: UI projects only.
- `docs/IMPLEMENTATION_PLAN_*.md`: the current work item.
- `docs/DECISIONS.md`: append-only durable decisions.
- `docs/MEMORY.md` and `docs/memory/YYYY-MM-DD.md`: dated execution memory.

## Repo Structure

```text
sdi-framework/
  README.md
  mvp-architect/
    SKILL.md
    references/
      core-templates/
      project-types/
      modifiers/
  convert-to-sdi/
    SKILL.md
    references/
  sdi-mode/
    MODE.md
    references/
    adapters/
      README.md
      roocode.md
      kilocode.md
      opencode.md
      cursor.md
      cline.md
      windsurf.md
      claudecode-codex/
        AGENTS.md
        CLAUDE.md
```

## What SDI Is Not

- **Not a code reviewer.** `sdi-mode` implements with discipline; the human still reviews the work.
- **Not an auto-approver.** Checkpoints require explicit review.
- **Not a refactor agent.** `convert-to-sdi` documents reality; it never edits source code.
- **Not scope-from-scratch for in-flight projects.** Use Phase E for next-plan work.
- **Not bureaucracy for its own sake.** Friction should earn its keep by improving clarity, quality, or auditability.

## Language

Source files are authored in English. Generated artifacts default to English for alignment with code, i18n, and model performance.

At runtime, the skills mirror the user's conversational language. If you start in Portuguese, they reply in Portuguese. If you want generated artifacts in another language, ask during generation.

## Contributing

Issues and PRs are welcome. Useful areas include:

- New project types, such as browser extensions, CLI tools, ML pipelines, or embedded projects.
- New modifiers beyond `ai.md`.
- Adapter improvements for additional tools.
- Documentation corrections and examples.

Contributions should follow the framework's own discipline: plan, audit, implement in reviewable rounds, and record decisions when they matter.

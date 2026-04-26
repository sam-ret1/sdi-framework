# SDI Framework

> **Spec-Driven Implementation** — a discipline for working with coding agents (Claude Code, Codex, Roo Code, Kilo Code, OpenCode) on software projects, from idea to shipped feature.

The framework is opinionated about *how* work is done, not *what* the product becomes. It provides three pieces — two skills for planning and one mode for execution — that turn a vague idea (or a legacy codebase) into a disciplined, auditable, reviewable engineering practice.

Stack-agnostic. Single-developer scale. Designed for the IDE-based AI coding workflow.

---

## The three pieces

| Piece | Type | Purpose | When |
|---|---|---|---|
| **`mvp-architect`** | Skill | Plan from idea to spec bundle. Review during implementation. Plan subsequent work items. | Starting greenfield, OR mid-implementation review/next-plan |
| **`convert2sdi`** | Skill | Adopt the framework on a project that already has code. | Joining a legacy project, or formalizing an in-flight greenfield |
| **`sdi-mode`** | Custom mode + `AGENTS.md` template | Execute work with audit-first discipline, binary-gate checkpoints, decisions log, daily memory. | Whenever code is being written against a spec |

The skills handle the *thinking*; the mode handles the *doing*. They were designed to compose.

---

## Quick start — which piece do I need?

```
                    Starting from a vague idea?
                              │
                              └──► mvp-architect (Phase 0 → A → B → C)
                                   Bundle generated. Then load sdi-mode.

                    Joining / adopting an existing codebase?
                              │
                              └──► convert2sdi (Phase 0 → 1 → 1.5 → 2 → 3)
                                   Audit + thin artifacts + first plan.
                                   Then load sdi-mode.

                    Implementing a planned work item?
                              │
                              └──► sdi-mode (via AGENTS.md or custom mode)

                    Mid-implementation, need review?
                              │
                              └──► mvp-architect Phase D
                                   (skip Phase 0/A/B/C, go straight to review)

                    Just finished a phase, need the next plan?
                              │
                              └──► mvp-architect Phase E
                                   (light, focused — reads current repo state)
```

---

## End-to-end flow

### Greenfield project (idea → shipped)

```
[Vague idea]
    │
    ▼
mvp-architect Phase 0 — pick project type (1 of 8) + AI modifier?
    │
    ▼
mvp-architect Phase A/B — discovery (universal + type-specific themes)
    │
    ▼
mvp-architect Phase C — generate bundle:
    PRD, ARCHITECTURE, ROADMAP, PROJECT_STRUCTURE, [DESIGN_SYSTEM],
    IMPLEMENTATION_PLAN_PHASE_1, AGENTS.md, README
    │
    ▼
[Project root: AGENTS.md + docs/ ready]
    │
    ▼
sdi-mode loaded → audit plan vs repo → implement Phase 1 in rounds
    │
    ▼
End of Phase 1: housekeeping
    │
    ├─► mvp-architect Phase D (review what was built, if needed)
    │
    └─► mvp-architect Phase E (plan Phase 2 / next slug)
                                    │
                                    ▼
                            sdi-mode implements next item
                                    │
                                    ▼
                                 (loop)
```

### Legacy project (existing code → adopted)

```
[Existing codebase, no framework artifacts yet]
    │
    ▼
convert2sdi Phase 0 — auto-audit (no questions)
    │
    ▼
convert2sdi Phase 1 — ≤5 triage questions ("não sei" always accepted)
    │
    ▼
convert2sdi Phase 1.5 — existing artifact handling (only if pre-existing
    docs detected). User picks A/B/C/D strategy per file.
    │
    ▼
convert2sdi Phase 2 — generate bundle (with confidence flags marking
    thin artifacts). Production-constraints section in AGENTS.md if
    project is in production.
    │
    ▼
convert2sdi Phase 3 — generate first IMPLEMENTATION_PLAN for the next
    concrete work item the user states.
    │
    ▼
[Project root: AGENTS.md + docs/ ready]
    │
    ▼
sdi-mode loaded → implement
    │
    ▼
mvp-architect Phase D ↔ Phase E loop continues from here
```

After convert2sdi runs once, **never run it again on the same project**. Subsequent work uses `mvp-architect` Phase E.

---

## Installation

### Claude Code

The skills live in your Claude Code skills directory:

- **Project-scoped:** `.claude/skills/mvp-architect/` and `.claude/skills/convert2sdi/`
- **User-scoped (global):** `~/.claude/skills/mvp-architect/` and `~/.claude/skills/convert2sdi/`

Copy the `mvp-architect/` and `convert2sdi/` directories from this repo to one of those locations.

For execution discipline, copy `sdi-mode/adapters/claudecode-codex/AGENTS.md` and `CLAUDE.md` to the root of any project where you want SDI mode active.

### Codex

Codex reads `AGENTS.md` natively from the project root. Copy `sdi-mode/adapters/claudecode-codex/AGENTS.md` to the root of your project.

The skills (`mvp-architect`, `convert2sdi`) don't apply directly — Codex doesn't have the skills system. Use them instead by running them in a Claude Code session, then move the generated artifacts to your Codex project. The AGENTS.md from this framework will then carry the SDI discipline in Codex sessions.

### Roo Code / Kilo Code / OpenCode

These tools don't have native skills but do support custom modes. Configure `sdi-mode` per the adapter for your tool:

- Roo Code: [`sdi-mode/adapters/roocode.md`](sdi-mode/adapters/roocode.md)
- Kilo Code: [`sdi-mode/adapters/kilocode.md`](sdi-mode/adapters/kilocode.md)
- OpenCode: [`sdi-mode/adapters/opencode.md`](sdi-mode/adapters/opencode.md)

The setup is similar across the three: paste `sdi-mode/MODE.md` into the mode's role/system-prompt, configure file restrictions on canonical artifacts, smoke-test.

For planning (mvp-architect, convert2sdi), use a separate Claude Code session and bring the resulting artifacts back into your Roo/Kilo/OpenCode workspace.

### Other tools (Cursor, Continue, Cody, Aider…)

Most modern coding agents respect either `AGENTS.md` or a tool-specific rules file (`.cursorrules`, `.continuerc`, etc.). Use the AGENTS.md template as a starting point and adapt to the tool's expected format.

---

## Project types covered

`mvp-architect` Phase 0 routes to one of eight type-specific playbooks. Each has its own discovery themes, architecture appendix, project structure, and (where applicable) design system:

1. **Web app SaaS multi-tenant** — Next.js/Remix/SvelteKit class with multi-org, billing, RLS
2. **Landing page / marketing site** — Astro/Next-static class with SEO, A/B, lead capture
3. **Dashboard / admin / internal tool** — data sources, queries, viz, RBAC
4. **Web API / backend service** — REST/GraphQL/RPC, webhooks, rate limiting
5. **Mobile app** — React Native, Flutter, native iOS/Android
6. **Data pipeline / scraper + reports** — ingestion → process → deliver
7. **AI agent / MCP server / chatbot** — LLM-driven, with tools, memory, eval, guardrails
8. **Integration / automation workflow** — Inngest/Temporal/n8n class

Plus: an **AI/LLM modifier** that adds cross-cutting concerns (prompts, eval, cost, RAG, guardrails) to *any* of the eight types when the project has significant LLM components.

Project not fitting any of the eight? The skill asks whether to proceed adapting on the fly or stop.

---

## The four core disciplines (SDI)

These are non-negotiable. They live in `sdi-mode/MODE.md` and propagate to every project via `AGENTS.md`.

1. **Audit the plan against the repo before coding.** The plan was written against assumptions; reality diverges. Catch divergences cheap, before they're baked into code. When plan disagrees with repo, **the repo wins**.

2. **Stop at explicit checkpoints with binary gate checklists.** Don't execute a whole phase end-to-end silently. Each phase has 2–5 natural checkpoints; every gate must pass before the round closes. Faking ticks is worse than failing them.

3. **Maintain `DECISIONS.md` (atemporal) and `memory/` (datable) as you go.** "Why did we pick this?" goes in DECISIONS — numbered, append-only. "What happened today / what's blocked / what's next" goes in `memory/YYYY-MM-DD.md`. Don't conflate.

4. **Respect document precedence.** When two docs disagree: live repo state > AGENTS.md > PRD > ARCHITECTURE > ROADMAP > PROJECT_STRUCTURE > IMPLEMENTATION_PLAN > DESIGN_SYSTEM > README. Lower doc gets a revision note. DECISIONS.md is overlay (patches), not authority. memory/ is breadcrumb, never source of truth.

---

## Living documents at project root

After running `mvp-architect` or `convert2sdi`, the project root carries living docs that the coding agent reads on every session:

- **`AGENTS.md`** — operating discipline + project-specific stack/conventions/work tracker. The framework's primary anchor at the project level. Updated as the project evolves.
- **`docs/MEMORY.md`** + **`docs/memory/YYYY-MM-DD.md`** — daily breadcrumb trail. What was worked on, blocked on, planned next.
- **`docs/DECISIONS.md`** — paper trail of non-obvious choices. Append-only, numbered.
- **`docs/IMPLEMENTATION_PLAN_*.md`** — current work item spec. `PHASE_N` for discrete linear phases (greenfield, migrations); `<slug>` for free-form work (features, maintenance) in ongoing projects.

Plus the static-ish ones generated once and updated at end-of-phase: `PRD.md`, `ARCHITECTURE.md`, `ROADMAP.md`, `PROJECT_STRUCTURE.md`, `DESIGN_SYSTEM.md` (UI projects only), `README.md`.

---

## Repo structure

```
sdi-framework/
├── README.md                             ← this file
│
├── mvp-architect/                        ← skill: plan from idea
│   ├── SKILL.md                          (6-phase flow: 0, A, B, C, D, E)
│   └── references/
│       ├── (cross-cutting refs)
│       ├── core-templates/               (universal: PRD, ROADMAP, etc.)
│       ├── project-types/                (8 type-specific dirs)
│       └── modifiers/
│           └── ai.md                     (AI/LLM cross-cutting)
│
├── convert2sdi/                          ← skill: adopt on legacy projects
│   ├── SKILL.md                          (3-phase flow + Phase 1.5 conditional)
│   └── references/                       (auto-audit, triage, artifact handling…)
│
└── sdi-mode/                             ← custom mode: execution discipline
    ├── MODE.md                           (canonical system prompt, stack-agnostic)
    ├── references/                       (audit protocol, round reports, etc.)
    └── adapters/
        ├── README.md                     (matrix: tool → adapter)
        ├── roocode.md
        ├── kilocode.md
        ├── opencode.md
        └── claudecode-codex/
            ├── AGENTS.md                 (template — copy to project root)
            └── CLAUDE.md                 (template — points to AGENTS.md)
```

---

## What the framework is NOT

- **Not a code reviewer.** sdi-mode implements with discipline; the human reviews the work. The framework explicitly resists rubber-stamping.
- **Not an auto-approver.** Stop-and-review checkpoints are binary gates. Silence is not consent.
- **Not a refactor agent.** `convert2sdi` documents reality; it never edits source code.
- **Not a scope-from-scratch tool for in-flight projects.** Use Phase E for next-plan, not Phase A/B/C.
- **Not bureaucracy.** If a step adds friction without increasing clarity, quality, or auditability, it should be questioned and redesigned — but never bypassed informally.

---

## Origin and inspiration

The framework was distilled from observation of how disciplined dev work with coding agents differs from "vibe coding". It draws structural inspiration from corporate agentic frameworks but operates at single-developer scale — no multi-agent orchestration, no central runtime, no telemetry overhead. Just disciplines that scale to one human + one coding agent + a series of work items.

The four core disciplines (audit-first, gates, DECISIONS+memory split, document precedence) emerged from watching agents drift on long-running projects: the audit catches assumption drift, the gates prevent silent end-runs, the memory split keeps both the "why" and the "where" searchable, and document precedence resolves the inevitable conflicts between artifacts at different layers.

---

## Language

Artifacts default to **English** (better alignment with code, easier i18n later, stronger model performance). Conversation can be in **PT-BR** (or other) — the framework is bilingual at the conversational layer. If you prefer artifacts in another language, just ask the skill at Phase C.

---

## Contributing

Issues and PRs welcome. Areas of natural growth:

- New project types (browser extensions, CLI tools, ML pipelines, embedded — currently out of scope but plausible additions)
- New modifiers (currently only `ai.md`; potential: realtime, integrations-heavy, marketplace)
- Adapter improvements (Cursor, Continue, Aider, etc.)
- Documentation improvements

Each contribution should follow the framework's own discipline — the skills + sdi-mode work on themselves. Yes, this framework can be (and was) used to plan changes to itself.

# SDI Mode — Adapters

`sdi-mode` is the implementation discipline (defined in [`MODE.md`](../MODE.md)). The adapters in this folder show how to load that discipline into specific coding-agent tools.

**Source of truth:** [`../MODE.md`](../MODE.md). Adapters reference it; they don't duplicate it.

## Tool matrix

| Tool | Adapter | Discipline carrier | Format |
|---|---|---|---|
| Claude Code | [`claudecode-codex/`](claudecode-codex/) | `AGENTS.md` (with `CLAUDE.md` pointer) | Markdown at repo root |
| Codex | [`claudecode-codex/`](claudecode-codex/) | `AGENTS.md` | Markdown at repo root |
| Roo Code | [`roocode.md`](roocode.md) | Custom mode in `.roomodes` | JSON config |
| Kilo Code | [`kilocode.md`](kilocode.md) | Custom mode | Tool-native config |
| OpenCode | [`opencode.md`](opencode.md) | Custom mode | Tool-native config |

## Which adapter to use

- **Claude Code or Codex:** copy [`claudecode-codex/AGENTS.md`](claudecode-codex/AGENTS.md) and [`claudecode-codex/CLAUDE.md`](claudecode-codex/CLAUDE.md) (Claude Code only) into the root of your project. Customize the placeholders for your project's type, stack, and conventions. The discipline loads automatically every session.

- **Roo Code:** follow [`roocode.md`](roocode.md) to register `sdi-mode` as a custom mode using the `.roomodes` config. After setup, switch to the mode for any implementation work.

- **Kilo Code:** follow [`kilocode.md`](kilocode.md). Same idea, Kilo's mode format.

- **OpenCode:** follow [`opencode.md`](opencode.md). Same idea, OpenCode's mode format.

- **Other tools (Cursor, Continue, Aider, Cody, etc.):** use Claude Code/Codex's `AGENTS.md` template as a starting point — most modern coding tools respect either `AGENTS.md` or a tool-specific `*.md` file. Adapt the file name and add tool-specific tweaks.

## What gets loaded vs what gets read on demand

**Always loaded** (system prompt / project root file):
- The 8-step discipline
- The 3 core rules (audit-first, stop-and-review, DECISIONS.md)
- "What this mode is not" (not reviewer, not auto-approver, not speculation engine)
- Tone guidelines

**Read on demand** (referenced when relevant):
- `references/audit-first-protocol.md` — when running an audit
- `references/round-report-template.md` — when writing a round report
- `references/decisions-log-format.md` — when adding a DECISIONS.md entry
- `references/revision-notes-format.md` — when revising a plan
- `references/stop-and-review-patterns.md` — when planning checkpoints
- `references/expected-artifacts.md` — when assessing handoff completeness

This split keeps the system prompt / project root file lean while making detail available when needed.

## File restrictions (recommended)

Most tools support file-edit restrictions per mode. Recommended restrictions for `sdi-mode`:

| File pattern | Restriction |
|---|---|
| `docs/PRD.md` | Edit only via revision note (don't rewrite content silently) |
| `docs/ARCHITECTURE.md` | Edit only via end-of-phase housekeeping |
| `docs/ROADMAP.md` | Edit only via end-of-phase housekeeping |
| `docs/PROJECT_STRUCTURE.md` | Edit during housekeeping or on convention discovery, with audit log |
| `docs/IMPLEMENTATION_PLAN_*.md` | Add revision notes only; never rewrite earlier content. Pattern matches both `PHASE_N` (discrete phases) and `<slug>` (free-form work) variants. |
| `docs/DECISIONS.md` | Append-only |
| `AGENTS.md` (root) | Update during phases, but propose changes explicitly to user |

The exact mechanism varies by tool; each adapter shows how.

## Verifying the mode is active

After setup, run a smoke check:

> "List the 8 steps of SDI discipline."

If the agent answers with the canonical list (Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping), the mode is active.

If the agent improvises or describes a different process, the discipline isn't loaded — re-check the configuration following the relevant adapter doc.

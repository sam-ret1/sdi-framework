# SDI Mode — Adapters

`sdi-mode` is the implementation discipline (defined in [`MODE.md`](../MODE.md)). The adapters in this folder show how to load that discipline into specific coding-agent tools.

**Source of truth:** [`../MODE.md`](../MODE.md). Adapters reference it; they don't duplicate it.

## Tool matrix

| Tool | Adapter | Discipline carrier | Format |
|---|---|---|---|
| Claude Code | [`claudecode-codex/`](claudecode-codex/) | `AGENTS.md` (with `CLAUDE.md` pointer) | Markdown at repo root |
| Codex | [`claudecode-codex/`](claudecode-codex/) | `AGENTS.md` | Markdown at repo root |
| Roo Code | [`roocode.md`](roocode.md) | Custom mode in `.roomodes` | YAML (or JSON) |
| Kilo Code | [`kilocode.md`](kilocode.md) | Agent in `.kilo/agents/sdi.md` (or `kilo.jsonc`) | Markdown + frontmatter (or JSON) |
| OpenCode | [`opencode.md`](opencode.md) | Agent in `.opencode/agents/sdi.md` (or `opencode.json`) | Markdown + frontmatter (or JSON) |
| Cursor | [`cursor.md`](cursor.md) | Project Rule in `.cursor/rules/sdi-mode.mdc` | Markdown + frontmatter |
| Cline | [`cline.md`](cline.md) | Workspace rule in `.clinerules/sdi-mode.md` | Markdown |
| Windsurf | [`windsurf.md`](windsurf.md) | Workspace rule in `.windsurf/rules/sdi-mode.md` | Markdown + frontmatter |

## Which adapter to use

- **Claude Code or Codex:** copy [`claudecode-codex/AGENTS.md`](claudecode-codex/AGENTS.md) and [`claudecode-codex/CLAUDE.md`](claudecode-codex/CLAUDE.md) (Claude Code only) into the root of your project. Customize the placeholders for your project's type, stack, and conventions. The discipline loads automatically every session.

- **Roo Code:** follow [`roocode.md`](roocode.md) to register `sdi-mode` as a custom mode in `.roomodes` (YAML preferred). After setup, switch to the mode for implementation work.

- **Kilo Code:** follow [`kilocode.md`](kilocode.md). Define an agent as `.kilo/agents/sdi.md` (markdown + YAML frontmatter — recommended) or as a JSON entry under the `agent` key in `kilo.jsonc`.

- **OpenCode:** follow [`opencode.md`](opencode.md). Define an agent as `.opencode/agents/sdi.md` or in `opencode.json`. The JSON config supports `prompt: "{file:./path/to/MODE.md}"`, which means MODE.md updates flow through automatically with no re-paste.

- **Cursor:** follow [`cursor.md`](cursor.md). Add `.cursor/rules/sdi-mode.mdc` with `alwaysApply: true`. The rule loads on every chat — no mode picker.

- **Cline:** follow [`cline.md`](cline.md). Add `.clinerules/sdi-mode.md`. Cline auto-merges all rule files in the directory; no frontmatter needed for unconditional always-on.

- **Windsurf:** follow [`windsurf.md`](windsurf.md). Add `.windsurf/rules/sdi-mode.md` with `activation: always_on`.

- **Other tools (Continue, Aider, GitHub Copilot Chat, Cody, etc.):** use Claude Code/Codex's `AGENTS.md` template as a starting point — most modern coding tools respect either `AGENTS.md` or a tool-specific `*.md` file. Quick adaptations:
  - **Continue.dev** — drop MODE.md as `.continue/rules/sdi-mode.md`, or define a custom mode with `systemMessage` in `.continue/config.yaml`.
  - **GitHub Copilot Chat** — copy MODE.md as `.github/copilot-instructions.md` (loaded automatically).
  - **Aider** — save MODE.md as `CONVENTIONS.md` and reference it via `read:` in `.aider.conf.yml`.

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

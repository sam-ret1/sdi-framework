# Cursor — SDI Mode adapter

This guide shows how to install `sdi-mode` as an always-applied rule in Cursor so the implementation discipline loads on every chat in the project.

## Prerequisites

- Cursor installed
- This `sdi-framework/` repo cloned or accessible
- A project where you want SDI mode active

## What you're configuring

Cursor doesn't have "custom modes" in the Roo/Kilo/OpenCode sense — instead, it has **Project Rules**: markdown files under `.cursor/rules/` that are loaded into the agent's context based on frontmatter. Cursor supports four rule types:

| Type | Frontmatter | Loaded |
|---|---|---|
| **Always Apply** | `alwaysApply: true` | Every chat in the project |
| **Apply Intelligently** | `alwaysApply: false` + `description` | When the agent decides it's relevant |
| **Apply to Specific Files** | `globs: [...]` | When edited files match the glob |
| **Manual** | (any) | Only when @-mentioned in chat |

For sdi-mode, **Always Apply** is the right choice — the discipline must be active for every implementation chat.

The legacy single-file `.cursorrules` at project root still works for backward compatibility, but Cursor recommends migrating to `.cursor/rules/*.mdc`.

## Step 1 — Create `.cursor/rules/sdi-mode.mdc`

```markdown
---
description: Spec-Driven Implementation discipline for the project — audit-first, stop-and-review, DECISIONS.md, round reports.
alwaysApply: true
---

<paste the full contents of sdi-framework/sdi-mode/MODE.md here>
```

The body of the file (everything after the closing `---`) becomes the rule content loaded into the agent's context. Markdown formatting is preserved.

**Why both `description` and `alwaysApply: true`?** When `alwaysApply: true`, the rule loads regardless of description. But the `description` is still shown in Cursor's UI and helps you (and your future self) recall what the rule does at a glance.

## Step 1 (alternative) — Legacy `.cursorrules`

If you're on an older Cursor version or want a single-file approach, create `.cursorrules` at the project root with this content:

```
<paste the full contents of sdi-framework/sdi-mode/MODE.md here>
```

No frontmatter; Cursor reads the whole file for every chat. Simpler but less flexible (no scoping, no companion rules per file pattern).

## Step 2 — Activate

1. Reload the Cursor window (or restart) so the rule is picked up.
2. No mode picker needed — `alwaysApply: true` rules load automatically on every chat.
3. Open Cursor's rules panel (Settings → Rules) to verify `sdi-mode.mdc` is listed and active.

## Step 3 — Smoke test

In a new chat, ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

If the agent improvises:
- Verify `.cursor/rules/sdi-mode.mdc` exists and parses (frontmatter is valid YAML)
- Check the rules panel shows it as active
- Confirm `alwaysApply: true` (without it, the rule might not load on every chat)

## File restrictions

Cursor rules don't have a hard-fence file-edit restriction equivalent to Roo/Kilo's regex `groups`. The framework's "PRD/ARCHITECTURE/ROADMAP edit only via revision notes" rule is enforced via the system prompt (soft fence). If you want a stricter setup, you can add a companion rule that re-emphasizes the canonical-artifact restrictions:

```markdown
---
description: Canonical artifact edit rules
globs: ["docs/PRD.md", "docs/ARCHITECTURE.md", "docs/ROADMAP.md", "docs/IMPLEMENTATION_PLAN_*.md"]
---

These files are canonical artifacts under the SDI framework. Do not rewrite them. For PRD / ARCHITECTURE / ROADMAP, only add revision notes at the top. For IMPLEMENTATION_PLAN_*, add `r2`, `r3`, etc. revision notes when reality diverges from the plan.
```

This rule loads only when the agent is editing a matching file, reinforcing the discipline at the moment of write.

## Splitting the discipline (optional)

`MODE.md` is ~190 lines. If you want to split it across multiple `.cursor/rules/*.mdc` files for clarity (the `alwaysApply: true` set), one possible split:

- `sdi-discipline.mdc` (always apply) — the 4 core rules + the 8-step loop
- `sdi-decisions.mdc` (always apply) — DECISIONS.md format and rules
- `sdi-memory.mdc` (always apply) — memory/ format and rules
- `sdi-on-demand.mdc` (manual / @-mention) — the references list with paths to load on demand

A single `sdi-mode.mdc` is fine for most projects; the split helps when you want to maintain different sections independently.

## Tips

- **Updating MODE.md:** when this framework updates `MODE.md`, re-paste the body of `.cursor/rules/sdi-mode.mdc`. Cursor doesn't currently support file-include in `.mdc`, so this is a copy step.
- **Project vs user scope:** `.cursor/rules/` is project-scoped (travels with the repo). Cursor also supports user-level rules in Settings → Rules → User Rules, useful when you want SDI mode active across all projects.
- **Plays well with `AGENTS.md`:** Cursor reads `AGENTS.md` at the repo root in addition to `.cursor/rules/`. If you've already copied [`claudecode-codex/AGENTS.md`](claudecode-codex/AGENTS.md) for Claude Code/Codex compatibility, Cursor will also pick it up — your `.cursor/rules/sdi-mode.mdc` and `AGENTS.md` will both load. This is fine; the duplication is harmless because both carry the same discipline.

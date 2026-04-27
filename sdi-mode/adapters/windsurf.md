# Windsurf — SDI Mode adapter

This guide shows how to install `sdi-mode` as a workspace rule in Windsurf so the implementation discipline loads in Cascade on every chat in the project. Windsurf also supports native `SKILL.md` skills; use those for `mvp-architect` and `convert-to-sdi` planning.

## Prerequisites

- Windsurf installed (Codeium's IDE)
- This `sdi-framework/` repo cloned or accessible
- A project where you want SDI mode active

## What you're configuring

Windsurf doesn't have "custom modes" in the Roo/Kilo/OpenCode sense — instead, Cascade reads **rules** from markdown files. Two scopes:

- **Workspace rules** — `.windsurf/rules/*.md` at the project root. Travels with the repo.
- **Global rules** — `~/.codeium/windsurf/memories/global_rules.md`, usually managed through the "Manage memories" / rules UI. Active across all workspaces.

Windsurf also reads `AGENTS.md` through the same rules engine: a root-level `AGENTS.md` is always on, and nested `AGENTS.md` files apply by directory. For SDI, `.windsurf/rules/sdi-mode.md` is still useful because it gives you Windsurf-native activation metadata.

## Step 1 — Create the rules directory

In your project root:

```
mkdir -p .windsurf/rules
```

## Step 2 — Add the SDI mode rule

Create `.windsurf/rules/sdi-mode.md`:

```markdown
---
description: Spec-Driven Implementation discipline. Active when implementing against an IMPLEMENTATION_PLAN_*.md.
trigger: always_on
---

<paste the full contents of sdi-framework/sdi-mode/MODE.md here>
```

The body of the file is the rule content loaded into Cascade's context. Markdown formatting is preserved.

**Frontmatter:**

| Field | Purpose |
|---|---|
| `description` | Shown in Windsurf's rules UI |
| `trigger` | `always_on` loads the rule on every chat. Other values include `model_decision` (Cascade decides based on description) and glob-based modes. |

## Step 3 — Activate

Windsurf picks up `.windsurf/rules/` automatically. To verify:

1. Open Cascade's "Manage memories" / rules panel.
2. Confirm `sdi-mode.md` is listed under workspace rules and is enabled.
3. Reload the workspace if it doesn't appear immediately.

## Step 4 — Smoke test

In a new Cascade chat, ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

If Cascade improvises:
- Open "Manage memories" and confirm the rule is enabled
- Verify the file body actually contains MODE.md content (not truncated)
- Reload the workspace

## Global rules option

If SDI is your default discipline across many projects, add it to `global_rules.md` instead (or in addition to) the workspace rule:

1. Cascade → Manage memories → Edit global rules
2. Append a section with MODE.md content

Workspace rules will still take precedence for any project that has its own `.windsurf/rules/sdi-mode.md`, so this works as a baseline that per-project rules can refine.

## Planning skills

Install `mvp-architect` and `convert-to-sdi` as native Windsurf skills:

- Project scope: `.windsurf/skills/mvp-architect/SKILL.md` and `.windsurf/skills/convert-to-sdi/SKILL.md`
- Global scope: `~/.codeium/windsurf/skills/mvp-architect/SKILL.md` and `~/.codeium/windsurf/skills/convert-to-sdi/SKILL.md`

Windsurf also discovers cross-agent skills in `.agents/skills/` and `~/.agents/skills/`. If Claude Code config reading is enabled, it can scan `.claude/skills/` and `~/.claude/skills/` too. Copy each whole skill directory from this repo, preserving its `SKILL.md` and `references/` folder.

Use the skills for planning and review. Use `.windsurf/rules/sdi-mode.md` for implementation discipline once a plan exists.

## File restrictions

Windsurf rules don't have a hard-fence file-edit restriction. The framework's "PRD/ARCHITECTURE/ROADMAP edit only via revision notes" rule is enforced via the system prompt (soft fence).

For a stricter Cascade setup, configure Cascade's auto-edit settings to require approval before writes — the discipline in MODE.md tells Cascade to add revision notes rather than rewrite, and Windsurf surfaces those edits for review.

## Token budget note

Windsurf has a rule budget. Workspace rule files are limited in size, and global rules have a smaller limit. If MODE.md plus your other workspace/global rules exceeds the available budget, Cascade may omit or truncate some rule content.

If that becomes an issue, split MODE.md into multiple rule files and let `description` + `model_decision` triggers pull in only what's relevant per chat. Possible split:

- `.windsurf/rules/sdi-core.md` (always_on) — the 4 core rules + the 8-step loop summary (~80 lines)
- `.windsurf/rules/sdi-decisions.md` (model_decision) — DECISIONS.md format
- `.windsurf/rules/sdi-memory.md` (model_decision) — docs/memory format
- `.windsurf/rules/sdi-references.md` (model_decision) — pointers to `sdi-framework/sdi-mode/references/`

For most projects, a single `.windsurf/rules/sdi-mode.md` with `trigger: always_on` is enough.

## Tips

- **Workspace rules are the right default** because the discipline ties to projects that have the SDI artifact bundle.
- **Updating MODE.md:** when this framework updates `MODE.md`, re-paste the body of `.windsurf/rules/sdi-mode.md`. Windsurf doesn't support file-includes in rules.
- **Plays well with `AGENTS.md`:** Windsurf can also read `AGENTS.md` if it's at the repo root. If you've already copied [`claudecode-codex/AGENTS.md`](claudecode-codex/AGENTS.md), Cascade will also pick it up — both will load. The duplication is harmless because both carry the same discipline.

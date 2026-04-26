# Cline — SDI Mode adapter

This guide shows how to install `sdi-mode` as a project rule in Cline so the implementation discipline loads on every chat in the project.

## Prerequisites

- Cline installed (VSCode extension)
- This `sdi-framework/` repo cloned or accessible
- A project where you want SDI mode active

## What you're configuring

Cline doesn't have "custom modes" in the Roo/Kilo/OpenCode sense — instead, it has **rules**: markdown files in a `.clinerules/` directory that Cline merges into the agent's context on every request.

Two scopes:

- **Workspace rules** — `.clinerules/` directory at the project root. Travels with the repo.
- **Global rules** — Cline's global rules folder (`~/Documents/Cline/Rules/` on macOS/Linux, similar on Windows). Available across all projects.

When both exist, Cline combines them, and **workspace rules take precedence on conflict**.

Cline processes all `.md` and `.txt` files inside the rules directory and combines them into a unified rule set. Practical performance starts to degrade past ~300 lines per file; aim for <150 for reliable rule adherence.

## Step 1 — Create the rules directory

In your project root:

```
mkdir -p .clinerules
```

## Step 2 — Add the SDI mode rule

Create `.clinerules/sdi-mode.md`:

```markdown
# SDI Mode — Implementation discipline

<paste the full contents of sdi-framework/sdi-mode/MODE.md here>
```

No frontmatter needed for unconditional always-loaded rules. The whole file is merged into Cline's context on every request.

> **MODE.md is ~190 lines** — within Cline's safe range. If you find Cline's rule adherence weakening, see "Splitting the discipline" below.

## Step 3 — Activate

Cline picks up `.clinerules/` automatically. To verify:

1. Open Cline's `.clinerules` popover (available in v3.13+) — it lists all active workspace and global rules with on/off toggles.
2. Confirm `sdi-mode.md` is listed and toggled on.
3. (Older Cline versions) reload the workspace if the rule isn't visible.

## Step 4 — Smoke test

In a new chat, ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

If the agent improvises:
- Open the `.clinerules` popover and confirm `sdi-mode.md` is on
- Verify the file body actually contains MODE.md content (not truncated)
- Reload the VSCode window if the rule was added during the session

## File restrictions

Cline doesn't support per-file edit restrictions at the rule level. The framework's "PRD/ARCHITECTURE/ROADMAP edit only via revision notes" rule is enforced via the system prompt (soft fence). For Cline's protection model:

- **Cline auto-approval settings** (in the chat UI) let you require approval per file edit. Setting "ask before every edit" is the closest hard fence available.
- The discipline in MODE.md tells the agent to add revision notes rather than rewriting these files; Cline will then surface those edits for your approval (which is what you want).

## Splitting the discipline (optional)

If you want to split MODE.md across multiple rule files for clarity or to stay well below the ~150-line guidance, one possible split:

- `.clinerules/sdi-discipline.md` — the 4 core rules + the 8-step loop (~80 lines)
- `.clinerules/sdi-decisions.md` — DECISIONS.md format and rules (~40 lines)
- `.clinerules/sdi-memory.md` — docs/memory format and rules (~40 lines)
- `.clinerules/sdi-references.md` — list of references in `sdi-framework/sdi-mode/references/` to load on demand

Cline merges all of these on every request, so functionally the result is identical to one big file.

## Conditional rules with frontmatter (advanced)

Cline supports YAML frontmatter on rules for conditional activation. If you want SDI mode to load only when an `IMPLEMENTATION_PLAN_*.md` exists in the repo, you can use frontmatter conditions; consult Cline's docs for the exact syntax. For most projects under sdi-framework, unconditional always-on is the right default — the discipline is meant to be active for every implementation session.

## Tips

- **Workspace `.clinerules/` is the right default** because the discipline ties to projects that have the SDI artifact bundle. A bare project (script, prototype) shouldn't carry SDI overhead.
- **Global rules folder** is useful for rules you want everywhere (e.g., personal coding preferences). Avoid putting sdi-mode there unless every project you work on is under the framework.
- **Updating MODE.md:** when this framework updates `MODE.md`, re-paste the body of `.clinerules/sdi-mode.md`. Cline doesn't support file-includes in rules.
- **Plays well with `AGENTS.md`:** Cline reads `AGENTS.md` at the repo root in addition to `.clinerules/`. If you've already copied [`claudecode-codex/AGENTS.md`](claudecode-codex/AGENTS.md), Cline will also pick it up — both will load. The duplication is harmless because both carry the same discipline.

# OpenCode — SDI Mode adapter

This guide shows how to install `sdi-mode` as a custom agent in OpenCode (sst/opencode).

## Prerequisites

- OpenCode installed
- This `sdi-framework/` repo cloned or accessible
- A project where you want SDI mode active

## What you're configuring

OpenCode supports two ways to define a custom agent — pick one:

1. **Markdown agent file** (recommended) — a `.md` file with YAML frontmatter; the body is the system prompt. Cleanest path for long prompts like MODE.md.
2. **JSON config** — `opencode.json` (project) or `~/.config/opencode/opencode.json` (global), under the `agent` key. Supports `{file:...}` references so you don't need to inline MODE.md.

Two scopes:

- **Project** — `.opencode/agents/sdi.md` or `.opencode/opencode.json`
- **Global** — `~/.config/opencode/agents/sdi.md` or `~/.config/opencode/opencode.json`

The filename of a markdown agent file becomes its identifier (e.g., `sdi.md` → `sdi` agent).

## Approach A — Markdown agent file (recommended)

Create `.opencode/agents/sdi.md`:

```markdown
---
description: Spec-Driven Implementation discipline. Use after planning is complete.
mode: primary
color: "#3b82f6"
permission:
  edit: ask
  bash:
    "*": ask
    "git status": allow
    "git diff*": allow
    "git log*": allow
  webfetch: ask
---

<paste the full contents of sdi-framework/sdi-mode/MODE.md here>
```

The body is the system prompt. Markdown formatting is preserved verbatim.

**Frontmatter fields used here:**

| Field | Purpose |
|---|---|
| `description` | Shown in the agent picker |
| `mode` | `primary` (user-selectable) / `subagent` (only callable from another agent) / `all` |
| `color` | UI color (hex or `primary`/`secondary`/`accent`/`success`/`warning`/`error`/`info`) |
| `permission.edit` | `allow` / `ask` / `deny` for file edits |
| `permission.bash` | Either a flat value or an object keyed by glob patterns |
| `permission.webfetch` | Network access |

Other available fields: `model`, `temperature`, `top_p`, `steps`, `hidden`, `disable`. Permission also supports `task` for subagent invocation control.

> **Note on file-edit restrictions.** OpenCode's `permission.edit` doesn't take per-glob deny lists like Kilo's permission system. The framework's "PRD/ARCHITECTURE/ROADMAP edit only via revision notes" rule is enforced via the system prompt (the soft fence) — there's no hard fence equivalent in OpenCode. If you want a hard fence, use `permission.edit: ask` so every write surfaces for approval.

## Approach B — JSON config (with `{file:...}` reference)

This is the cleanest way to avoid copying MODE.md content. Create or edit `opencode.json` at the project root:

```json
{
  "agent": {
    "sdi": {
      "description": "Spec-Driven Implementation discipline. Use after planning is complete.",
      "mode": "primary",
      "color": "#3b82f6",
      "prompt": "{file:./sdi-framework/sdi-mode/MODE.md}",
      "permission": {
        "edit": "ask",
        "bash": {
          "*": "ask",
          "git status": "allow",
          "git diff*": "allow",
          "git log*": "allow"
        },
        "webfetch": "ask"
      }
    }
  }
}
```

The `{file:...}` syntax loads the prompt from disk at config-parse time. Path is relative to the config file location. **This means MODE.md updates are picked up automatically on the next OpenCode reload — no re-paste required.**

If you don't have `sdi-framework/` cloned inside your project, point the path at wherever you keep the repo, e.g. `{file:/Users/me/repos/sdi-framework/sdi-mode/MODE.md}` (absolute) or `{file:../sdi-framework/sdi-mode/MODE.md}` (relative).

For a global config, use `~/.config/opencode/opencode.json` with the same `agent.sdi` schema.

## Step 2 — Activate

1. Reload OpenCode (restart or use the reload command).
2. Select `sdi` in the agent picker.
3. Confirm the agent badge shows it's active.

## Step 3 — Smoke test

Ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

If the agent improvises:
- Verify the file exists at `.opencode/agents/sdi.md` (Approach A) or that the `{file:...}` path resolves (Approach B)
- Check that the agent is selected (not the default)
- Confirm config syntax is valid (no parse errors in OpenCode's output)

## Tips

- **`{file:...}` is a killer feature.** With Approach B, MODE.md updates flow through automatically — no copy-paste cycle. This is the recommended setup if you have `sdi-framework/` checked out as a sibling or sub-directory of your project.
- **Multiple agents.** When you set up `mvp-architect` / `convert2sdi` analogues later in OpenCode, define each as a separate `.md` file under `.opencode/agents/` or as another entry under the `agent` object in JSON.
- **Updating MODE.md:** Approach A → re-paste the body and reload. Approach B → no action; reload picks up the new file content.
- **Permission tuning:** `bash: { "*": "ask" }` is the safest default. As trust grows on a project, you can add `"npm test": "allow"`, `"npm run *": "allow"`, etc.

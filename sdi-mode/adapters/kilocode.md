# Kilo Code — SDI Mode adapter

This guide shows how to install `sdi-mode` as a custom agent (Kilo's term for a custom mode) in Kilo Code.

## Prerequisites

- Kilo Code installed
- This `sdi-framework/` repo cloned or accessible (you'll reference [`MODE.md`](../MODE.md) from the agent file)
- A project where you want SDI mode active

## What you're configuring

Kilo Code (originally a Roo Code fork) has converged on an OpenCode-compatible **agent** model. There are two ways to define an agent — pick one:

1. **Markdown agent file** (recommended) — a `.md` file with YAML frontmatter; the body becomes the system prompt. Cleanest because there's no escaping.
2. **JSON config** — `kilo.jsonc` at project root or `.kilo/kilo.jsonc` (the latter has priority if both exist).

Kilo can also read `AGENTS.md` from the project root. Use that for project-specific conventions and tool parity; use the dedicated `sdi` agent when you want explicit agent selection, prompt isolation, and permission defaults for SDI implementation sessions.

Two scopes:

- **Project** — `.kilo/agents/sdi.md` or `kilo.jsonc` / `.kilo/kilo.jsonc` at project root.
- **Global** — `~/.config/kilo/agent/sdi.md` or `~/.config/kilo/kilo.jsonc`. (On Windows: `C:\Users\<user>\.config\kilo\...`)

## Approach A — Markdown agent file (recommended)

Create `.kilo/agents/sdi.md` in your project (or `~/.config/kilo/agent/sdi.md` for global):

```markdown
---
description: Spec-Driven Implementation discipline. Use after planning is complete.
mode: primary
color: "#3b82f6"
permission:
  edit:
    "docs/PRD.md": deny
    "docs/ARCHITECTURE.md": deny
    "docs/ROADMAP.md": deny
    "*": allow
  bash: ask
---

<paste the full contents of sdi-framework/sdi-mode/MODE.md here>
```

The body of the file (everything after the closing `---`) becomes the system prompt. Markdown formatting is preserved.

**Frontmatter fields used here:**

| Field | Purpose |
|---|---|
| `description` | Shown in the agent picker |
| `mode` | `primary` (selectable as the active agent) / `subagent` / `all` |
| `color` | UI color in agent picker (hex or named keyword) |
| `permission.edit` | Per-glob file-edit restrictions; `deny` blocks the listed canonical artifacts |
| `permission.bash` | `allow` / `ask` / `deny` for shell commands |

Other available frontmatter fields: `model`, `temperature`, `top_p`, `steps`, `hidden`, `disable`. See Kilo's docs for the complete list.

## Approach B — JSON config (`kilo.jsonc`)

If you prefer keeping all agents in one config file, create `.kilo/kilo.jsonc` (or `kilo.jsonc` at project root):

```jsonc
{
  "agent": {
    "sdi": {
      "description": "Spec-Driven Implementation discipline. Use after planning is complete.",
      "mode": "primary",
      "color": "#3b82f6",
      "prompt": "<JSON-escaped contents of MODE.md, OR a {file:./path/to/MODE.md} reference if your version supports it>",
      "permission": {
        "edit": {
          "docs/PRD.md": "deny",
          "docs/ARCHITECTURE.md": "deny",
          "docs/ROADMAP.md": "deny",
          "*": "allow"
        },
        "bash": "ask"
      }
    }
  }
}
```

**Caveat:** the `prompt` field needs JSON-escaped newlines (`\n`). For long content like MODE.md, the markdown agent file approach (A) is much easier — that's why it's recommended.

## Step 2 — Activate

1. Reload Kilo Code (or restart) so the agent config is picked up.
2. Open Kilo's agent/mode selector.
3. Choose `sdi`.
4. Verify the agent badge shows it's active.

## Step 3 — Smoke test

Ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

If the agent improvises:
- Verify the agent file path (`.kilo/agents/sdi.md` or the JSON config path) is correct
- Check that the markdown body / `prompt` field actually contains MODE.md content (not truncated)
- Confirm `sdi` is the selected agent (not the default)

## File restrictions

The `permission.edit` block enforces the framework's "PRD/ARCHITECTURE/ROADMAP edit only via revision notes" rule as a hard fence. The system prompt's discipline is the soft fence — both reinforce the same behavior.

To also block `IMPLEMENTATION_PLAN_*.md` from full rewrites:

```yaml
permission:
  edit:
    "docs/PRD.md": deny
    "docs/ARCHITECTURE.md": deny
    "docs/ROADMAP.md": deny
    "docs/IMPLEMENTATION_PLAN_*.md": ask
    "*": allow
```

Glob `*` matches the slug or `PHASE_N` part. `ask` requires explicit user approval per edit, which suits "revision notes only" semantics — the agent has to surface what it's about to change.

## Tips

- **Markdown approach scales.** When you add `mvp-architect` / `convert-to-sdi` style agents later, define them as additional `.kilo/agents/*.md` files — same pattern, different `description` and frontmatter.
- **Global vs project:** per-project `.kilo/agents/sdi.md` travels with the repo. Global `~/.config/kilo/agent/sdi.md` is convenient when SDI is your default discipline everywhere.
- **Updating MODE.md:** with the markdown approach, just re-paste the body. With the JSON approach, re-escape the string. Either way, reload Kilo afterward.
- **Configuration precedence (lowest → highest):** built-in defaults → global config → project `kilo.jsonc` → `.kilo/agents/*.md` files. Project markdown agents win over JSON config; useful if you keep a global JSON baseline and override per-project.

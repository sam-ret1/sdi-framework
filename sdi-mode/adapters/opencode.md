# OpenCode — SDI Mode adapter

This guide shows how to install `sdi-mode` as a custom mode (or its equivalent) in OpenCode.

## Prerequisites

- OpenCode installed
- This `sdi-framework/` repo cloned or accessible
- A project where you want SDI mode active

## What you're configuring

OpenCode supports custom agent configurations via its config files (typically under `.opencode/` in the workspace, or in the global config). The exact field names vary by OpenCode version, but the pattern is consistent:

- A named mode/agent
- A system prompt (where MODE.md content goes)
- Tool / permission restrictions
- Model selection

Consult OpenCode's current docs for the exact schema if it has changed since this guide was written. The conceptual steps below are stable.

## Step 1 — Create the mode config

In your workspace, create `.opencode/agents/sdi.json` (or the equivalent in your OpenCode version):

```json
{
  "name": "SDI — Implementation",
  "description": "Spec-Driven Implementation discipline. Use after planning is complete.",
  "model": "claude-sonnet-4-6",
  "systemPrompt": "<paste the contents of sdi-framework/sdi-mode/MODE.md here>",
  "tools": ["read", "edit", "bash"],
  "fileRestrictions": {
    "editDeny": [
      "docs/PRD.md",
      "docs/ARCHITECTURE.md",
      "docs/ROADMAP.md"
    ],
    "editDenyPattern": "docs/IMPLEMENTATION_PLAN_[\\w-]+\\.md"
  }
}
```

Adjust:
- `model` — your preferred model for implementation work
- `tools` — the toolset the agent has access to
- `fileRestrictions.editDeny` — files the agent cannot edit directly (only via revision notes); add `IMPLEMENTATION_PLAN_*` if desired

If your OpenCode version doesn't support file-deny lists, the system prompt's discipline rules cover the soft fence.

## Step 2 — Paste MODE.md content

Open [`MODE.md`](../MODE.md). Copy its full content into the `systemPrompt` field. Markdown formatting is OK; JSON-escape newlines as `\n` if your config requires it (some YAML/TOML alternatives let you use multi-line strings natively).

## Step 3 — Activate

1. Reload OpenCode (restart or use the reload command).
2. Select `SDI — Implementation` in the mode/agent picker.
3. Confirm the mode badge / indicator shows it's active.

## Step 4 — Smoke test

Ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

If the agent improvises, the system prompt didn't load. Check:
- Config file syntax is valid (no parse errors)
- The mode is actually selected
- `systemPrompt` contains MODE.md content (not truncated)

## Tips

- **Reference loading:** OpenCode may support per-mode "knowledge" or "reference" files. If so, point them at `sdi-framework/sdi-mode/references/` so audit/round-report formats are loadable on demand.
- **Updating MODE.md:** re-paste when the framework updates the mode source.
- **Multi-project use:** prefer global config when SDI is your default discipline; per-workspace config when you have project-specific tweaks.

## When OpenCode doesn't support custom modes natively

Some OpenCode versions / forks don't support fully custom modes. Fallback options:

1. **Use AGENTS.md instead:** OpenCode reads `AGENTS.md` at repo root by convention. Use the [`claudecode-codex/AGENTS.md`](claudecode-codex/AGENTS.md) template as your loading surface.
2. **Embed in a project rules file** if OpenCode supports per-project rule files (similar to `.cursorrules`).
3. **Use a hook or initial system prompt:** copy MODE.md content into the first message of a chat as system context; less elegant but functional.

The discipline doesn't depend on a specific delivery mechanism — it depends on the content being in the agent's working context throughout the session.

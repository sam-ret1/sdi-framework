# Kilo Code — SDI Mode adapter

This guide shows how to install `sdi-mode` as a custom mode in Kilo Code.

## Prerequisites

- Kilo Code installed
- This `sdi-framework/` repo cloned or accessible (you'll copy [`MODE.md`](../MODE.md) content into the mode config)
- A project where you want SDI mode active

## What you're configuring

Kilo Code (a Roo Code fork) uses a similar custom-mode model. Modes live in workspace or global config and define:

- Slug + display name
- System prompt / role definition (this is where MODE.md goes)
- Tool / file-edit restrictions
- Model selection

## Step 1 — Open Kilo Code custom modes config

Depending on Kilo Code version, custom modes are defined in:

- `.kilocode/modes.json` (workspace-scoped) — preferred for project-specific
- Global settings via the Kilo Code UI — for a mode you want everywhere

Create or open the relevant file.

## Step 2 — Add the SDI mode

```json
{
  "customModes": [
    {
      "slug": "sdi",
      "name": "SDI — Implementation",
      "roleDefinition": "<paste the contents of sdi-framework/sdi-mode/MODE.md here>",
      "groups": [
        "read",
        ["edit", { "fileRegex": "^(?!docs/(PRD|ARCHITECTURE|ROADMAP)\\.md$).*", "description": "PRD/ARCHITECTURE/ROADMAP edited only via revision notes" }],
        "command"
      ]
    }
  ]
}
```

Same JSON-string caveat as Roo Code: newlines in `roleDefinition` need escaping, or use a file-loading feature if your Kilo Code version supports it.

## Step 3 — Paste MODE.md content

Open [`MODE.md`](../MODE.md). Copy its full content into the `roleDefinition` field. Make sure the markdown formatting survives (escape only what JSON requires).

## Step 4 — Activate

1. Reload the workspace (or restart Kilo Code) so the mode config is picked up.
2. Open Kilo Code's mode selector.
3. Choose `SDI — Implementation`.
4. Verify mode is active.

## Step 5 — Smoke test

Ask:

> "List the 8 steps of SDI discipline."

Expected: Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping.

## Tips

- **Updates to MODE.md:** re-paste when this framework updates MODE.md, then reload.
- **Per-project vs global:** projects in early phases benefit from local `.kilocode/modes.json` (specific to one stack); global settings are convenient when SDI is your default discipline across all projects.
- **Mode-switch ritual:** when leaving implementation for scoping, switch out of SDI mode. The discipline is for execution work, not exploratory thinking.

## File restrictions

Same recommendation as Roo Code: prevent silent rewrites of PRD / ARCHITECTURE / ROADMAP. Use:

```json
"fileRegex": "^(?!docs/(PRD|ARCHITECTURE|ROADMAP)\\.md$).*"
```

Or, stricter (also blocks `IMPLEMENTATION_PLAN_*` edits — both `PHASE_N` and `<slug>` variants):

```json
"fileRegex": "^(?!docs/(PRD|ARCHITECTURE|ROADMAP|IMPLEMENTATION_PLAN_[\\w-]+)\\.md$).*"
```

Choose based on how strict you want the soft-vs-hard fence balance.

## Differences from Roo Code

If you've used Roo Code's `.roomodes`, Kilo Code is largely the same model with cosmetic differences:

- File location may differ (`.kilocode/` vs `.roomodes`)
- UI flow for mode selection may differ slightly
- Some advanced features (tool restrictions beyond file-edit) may differ

The core idea — "paste MODE.md as roleDefinition, set file restrictions, smoke test" — is identical.

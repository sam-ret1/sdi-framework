# Roo Code — SDI Mode adapter

This guide shows how to install `sdi-mode` as a custom mode in Roo Code so the implementation discipline loads on every session.

## Prerequisites

- Roo Code installed (VSCode extension)
- This `sdi-framework/` repo cloned somewhere accessible (you'll reference [`MODE.md`](../MODE.md) by path or copy its content into the mode config)
- A project where you want SDI mode active

## What you're configuring

Roo Code custom modes live in `.roomodes` (JSON file at the workspace root) or in the global Roo Code settings. Each mode has:

- An identifier and display name
- A "role definition" / system prompt — this is where MODE.md content goes
- Optional file-edit restrictions
- Optional model preference

## Step 1 — Create or edit `.roomodes`

In the root of your project, create or edit `.roomodes`:

```json
{
  "customModes": [
    {
      "slug": "sdi",
      "name": "SDI — Implementation",
      "roleDefinition": "<paste the contents of sdi-framework/sdi-mode/MODE.md here>",
      "groups": [
        "read",
        ["edit", { "fileRegex": "^(?!docs/(PRD|ARCHITECTURE|ROADMAP)\\.md$).*", "description": "Cannot directly rewrite PRD, ARCHITECTURE, or ROADMAP — use revision notes" }],
        "command",
        "browser"
      ]
    }
  ]
}
```

Notes on the regex above: it allows edits to all files **except** `docs/PRD.md`, `docs/ARCHITECTURE.md`, and `docs/ROADMAP.md`. Adjust to taste; a stricter version also restricts `IMPLEMENTATION_PLAN_PHASE_*.md` to revision-note edits only, but Roo's regex group syntax doesn't natively express "edit only via append" — you may rely on the system prompt to enforce the discipline.

## Step 2 — Paste MODE.md content

Open [`MODE.md`](../MODE.md) in this framework. Copy its full content into the `roleDefinition` field of the JSON above.

Tip: since JSON doesn't allow raw newlines in strings, either:
- Use a Roo Code feature that supports loading roleDefinition from a file path (if available in your version), or
- Convert MODE.md content to a single JSON-escaped string (newlines as `\n`)

## Step 3 — Add reference shortcuts (optional)

Roo Code supports per-mode commands and prompts. Consider adding shortcuts that load specific references on demand:

```json
"customInstructions": "When the user says 'audit', read sdi-framework/sdi-mode/references/audit-first-protocol.md and follow that format. When the user says 'round report', read sdi-framework/sdi-mode/references/round-report-template.md."
```

## Step 4 — Activate

In Roo Code:

1. Reload the workspace so `.roomodes` is picked up.
2. Open the mode selector (usually in the chat header).
3. Choose `SDI — Implementation`.
4. Verify the system prompt indicator shows the mode name.

## Step 5 — Smoke test

Ask:

> "List the 8 steps of SDI discipline."

Expected response: the canonical list (Read → Audit → Propose → Implement in rounds → Tests alongside → DECISIONS.md → Revision notes → End-of-phase housekeeping).

If the agent improvises, re-check that:
- `.roomodes` was loaded (no JSON parse errors in the Roo Code output panel)
- The mode is selected (not "Code" or default)
- `roleDefinition` actually contains MODE.md content (not a truncated copy)

## Tips

- **One mode per workspace** is fine. You don't need to nest `sdi-mode` inside other modes.
- **Mode switching:** when you finish implementation work and want to scope something new, switch to the default Code mode or a custom `mvp-architect` equivalent. SDI mode's discipline is for execution, not scoping.
- **Updating MODE.md:** when this framework updates `MODE.md`, re-paste into `.roomodes` and reload.
- **Multi-project use:** if you want SDI mode globally available, register it in Roo Code's global settings instead of per-workspace `.roomodes`.

## File restrictions detail

The recommended file-edit restriction prevents the agent from silently rewriting PRD/ARCHITECTURE/ROADMAP. The discipline allows revision notes (small additions at the top) but not full rewrites. Roo Code's regex restriction is a hard fence; the system prompt's instruction to "use revision notes" is the soft fence. Both reinforce the same rule.

If you want a stricter pattern, also block `docs/IMPLEMENTATION_PLAN_*.md` (both `PHASE_N` and `<slug>` variants):

```json
"fileRegex": "^(?!docs/(PRD|ARCHITECTURE|ROADMAP|IMPLEMENTATION_PLAN_[\\w-]+)\\.md$).*"
```

The `[\\w-]+` charclass matches `PHASE_1`, `PHASE_2`, `billing-portal`, `q1-perf-pass`, etc. — any of the framework's plan naming variants.

…but be aware the agent will then need to ask permission before adding a revision note to a plan, which is friction. Choose based on how much the agent has earned trust on this project.

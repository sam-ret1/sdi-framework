# Existing artifact handling

When `convert-to-sdi` runs against a project that already has artifacts (`PRD.md`, `ARCHITECTURE.md`, `ROADMAP.md`, ADRs, etc.) — possibly in formats different from the framework's canonical structure — the skill must NOT silently overwrite them. This reference defines:

1. The four canonical **strategies** for handling existing artifacts
2. The **default strategy per artifact** (a matrix the skill applies)
3. The **decision flow** the skill runs in **Phase 1.5** (between Phase 1 triage and Phase 2 generation)
4. **Special cases** (ADRs, in-tool docs like Notion/Figma/Linear, custom `docs/` folders)

The whole point: never lose user content, never silently fight with their conventions, never duplicate without clear reason.

## Contents

- [The four strategies](#the-four-strategies) — A/B/C/D handling options for existing artifacts
- [The matrix — default strategy per artifact](#the-matrix--default-strategy-per-artifact) — what to do per file type
- [ADR handling (special case)](#adr-handling-special-case) — preserve ADR investment, default Option 1
- [In-tool / external docs](#in-tool--external-docs-notion-linear-figma-confluence) — Notion, Linear, Figma, Confluence handling
- [Custom `docs/` content](#custom-docs-content) — runbooks, post-mortems, internal wiki
- [Format compatibility detection (heuristic)](#format-compatibility-detection-heuristic) — per-artifact compatibility classification
- [Phase 1.5 decision flow](#phase-15-decision-flow) — present matrix → user respond → confirm
- [Strategy execution rules](#strategy-execution-rules) — how each strategy is applied in Phase 2
- [Don't-do list](#dont-do-list) — rules to avoid breaking user content
- [Recovery if convert-to-sdi was already run badly](#recovery-if-convert-to-sdi-was-already-run-badly) — git-based safety net

## The four strategies

### Strategy A — Preserve as-is + framework header

The existing file already does its job. Don't touch the content. Add a small header at the top so future readers know the framework is active.

**Pattern:**
```markdown
> **Framework:** This project uses the sdi-framework workflow. See `AGENTS.md` (root) for operating discipline. This document predates the framework adoption and is preserved as-is.

[existing content unchanged]
```

**When:**
- Existing file already covers the canonical role (README with stack + summary + doc links; AGENTS.md already in SDI format)
- Format compatibility is high (matches framework structure or close enough)
- User chose to keep current format (e.g. team has muscle memory for it)

**Cost:** zero rework, zero risk.

### Strategy B — Convert in place

Read the existing content. Propose a new version that **merges** canonical framework structure + existing content. Show the user the diff. They approve, edit, or reject.

**Pattern:**
1. Skill reads existing file.
2. Skill identifies what canonical sections are present, partially present, or missing.
3. Skill writes a new version: existing content reorganized into canonical sections; missing sections added as placeholders with confidence flags; extra content from the existing file kept as a `## Legacy notes` section at the end (or distributed where it fits).
4. Skill presents the diff to the user before writing.
5. User says: "go", "edit X", or "keep original".

**When:**
- Existing file has substance worth preserving but format diverges
- User wants framework-canonical structure going forward
- Format mismatch isn't extreme

**Cost:** ~5 min user review per file. Diff approval should be quick if the conversion is clean.

### Strategy C — Rename existing + generate new

Rename the existing file to `<name>.legacy.md` (or `<name>.original.md`). Generate a fresh canonical version from the framework template.

**Pattern:**
1. Skill renames `docs/ROADMAP.md` → `docs/ROADMAP.legacy.md`.
2. Skill generates fresh `docs/ROADMAP.md` from the framework template.
3. Skill adds a one-line note at the top of the new file: *"Predecessor preserved at [`ROADMAP.legacy.md`](./ROADMAP.legacy.md). Reference for historical context; not the source of truth."*
4. Skill mentions the legacy file in today's `docs/memory/YYYY-MM-DD.md` entry.

**When:**
- Existing file is in a fundamentally incompatible format (e.g. ROADMAP that's actually a Trello CSV export, ARCHITECTURE that's a 3000-line wiki dump)
- User explicitly wants a clean canonical version
- Existing is too outdated to merge usefully

**Cost:** zero rework on existing (preserved), but two files now exist — slight maintenance overhead.

### Strategy D — Skip generation entirely

Don't generate the canonical artifact at all. Add a note in `AGENTS.md` (and optionally `README.md`) explaining where the source of truth lives.

**Pattern in AGENTS.md:**
```markdown
## External artifact references

Some framework artifacts live outside this repo:

- **PRD:** Notion page [link]. The framework-canonical PRD.md is intentionally not generated; treat the Notion page as authoritative.
- **Roadmap:** Linear cycles + project view [link]. ROADMAP.md is not generated; Linear is the source of truth.
- **Design system:** Figma file [link] + the implemented tokens in `src/styles/tokens.css`. DESIGN_SYSTEM.md is not generated.

When sdi-mode references PRD.md / ROADMAP.md / DESIGN_SYSTEM.md, substitute the external source above.
```

**When:**
- Existing source of truth lives in another tool (Notion, Linear, Jira, Figma, Confluence)
- User explicitly wants to keep working in that tool, not migrate to markdown
- Forcing migration would create duplication and drift

**Cost:** sdi-mode loses some on-repo context for that artifact, but the user controls trade-off explicitly.

## The matrix — default strategy per artifact

Skill applies these defaults; user can override per-file in Phase 1.5.

| Artifact | If exists, default strategy | Conditions for override |
|---|---|---|
| `README.md` | **A** (preserve + framework header) | B if it lacks stack table + doc links AND user wants alignment |
| `AGENTS.md` | **A** if SDI-formatted; **B** if framework-aware but partial | C if user wants fresh |
| `PROJECT_STRUCTURE.md` | **B** (existing as basis, add canonical sections) | A if already canonical |
| `ARCHITECTURE.md` | **B** (merge with type appendix sections) | C if outdated >1 year or wiki-style |
| `PRD.md` | **B** (preserve content, fit into canonical sections) | D if PRD lives in Notion/Confluence |
| `ROADMAP.md` | **B** if has phase/quarter structure; **C** if pure export | D if in Linear/Jira |
| `DESIGN_SYSTEM.md` | **B** (extract tokens, structure into canonical) | D if living in Figma |
| `DECISIONS.md` (single file) | **B** (merge into framework format if needed) | A if format already matches |
| `docs/decisions/` or `docs/adr/` (ADR folder) | **Special — see ADR handling below** | |
| `CHANGELOG.md` | **A** (preserve; signals are channeled to AGENTS production-indicators section) | never overwrite |
| Custom `docs/` files (runbooks, post-mortems, internal wiki pages) | **A** (preserve all; reference where relevant from AGENTS doc map) | |
| `.cursorrules`, `.continuerc`, `.windsurfrules`, IDE-specific rule files | **A** (preserve; surface as "alternative discipline carriers" in AGENTS) | propose deprecation if redundant with AGENTS |

**Default safety bias:** when in doubt, prefer A or B over C. C is destructive of organization (renames file). D is conservative but creates external dependency.

## ADR handling (special case)

ADRs (Architecture Decision Records) are a mature, widely-adopted convention — typically `docs/decisions/0001-decision-name.md`, `0002-...`, etc. Don't fight ADR investment. Two acceptable approaches:

### Option 1 — Keep ADRs, generate DECISIONS.md as an index (DEFAULT)

```markdown
# DECISIONS

> **Source:** index pointing to ADR records in `docs/decisions/`. The ADR convention pre-dates the framework adoption; this file aggregates them and adds new framework-style entries below.

## ADR records (authoritative)

| # | Title | File |
|---|---|---|
| 0001 | [title from ADR file] | [docs/decisions/0001-...](docs/decisions/0001-...) |
| 0002 | ... | ... |

## Framework-style additions

(New decisions taken under SDI mode are added below in framework format. ADRs continue to be the canonical record for architecture-level decisions; framework-style entries cover implementation-level decisions per work item.)
```

**Why default:** preserves ADR investment + adds framework's lighter format for implementation decisions. Both coexist. Convention: architectural decisions → ADR; implementation decisions → DECISIONS.md.

### Option 2 — Convert ADRs to DECISIONS.md format

Read each ADR, produce a framework-style entry in `DECISIONS.md`, preserve the ADR number as a marker:

```markdown
### #ADR-0001 — [original title]

**Context**: [extracted from ADR §Context]

**Decision**: [extracted from ADR §Decision]

**Rationale**: [extracted from ADR §Rationale or §Consequences]

**Source**: converted from `docs/decisions/0001-...md` on YYYY-MM-DD.

**Revisit when**: [extracted if ADR has it; "?" otherwise]
```

After conversion, rename ADR folder to `docs/decisions.legacy/` (Strategy C applied to the folder).

**When:** user explicitly wants single-format DECISIONS and is willing to give up ADR convention. **Rare.**

The skill **always defaults to Option 1**. Option 2 only if the user explicitly asks.

## In-tool / external docs (Notion, Linear, Figma, Confluence)

When existing source-of-truth lives outside the repo, **do not import or migrate**. Apply Strategy D and document the external pointer in AGENTS.md.

Common patterns:

| Artifact | Common external tool | Treatment |
|---|---|---|
| PRD | Notion, Confluence, Coda | D + link in AGENTS.md "External artifact references" |
| Roadmap | Linear cycles, Jira epics, ClickUp | D + link |
| Design system | Figma, Storybook deployed | D + link; tokens may still be extractable from code |
| Runbooks | Notion, internal wiki | A (if in repo) or D (if external) |
| Customer docs | Help Center, Intercom | D |

The skill asks in Phase 1.5: *"Some artifacts may live in tools (Notion, Linear, Figma, etc.). Should any of these stay external? List the ones, with links."*

## Custom `docs/` content

Many projects accumulate `docs/`-folder content that's neither canonical framework artifact nor pure ADR — runbooks, post-mortems, integration guides, onboarding docs, etc.

Default: **Strategy A — preserve all**. The skill scans `docs/` and:

1. Lists all non-canonical files in the discovery report
2. Asks user (in Phase 1.5) if any should be linked from AGENTS.md "Document map" → "Other docs" section
3. Default-links all of them under a flat list

Don't reorganize the `docs/` folder unless the user asks. Don't delete anything.

## Format compatibility detection (heuristic)

To decide between A (preserve) and B (convert in place), the skill checks **format compatibility** during Phase 0 audit. Heuristics per artifact:

### `README.md`
**Compatible** if has all of: 1-paragraph product summary, stack table or stack list, links to other docs.
**Partially compatible**: has 2 of the 3.
**Incompatible**: just title + clone instructions; no stack/docs.

### `PRD.md`
**Compatible** if has identifiable sections matching: product/goal description, users/audience, in-scope features, out-of-scope features, success metrics.
**Partially compatible**: has 3+ of 5.
**Incompatible**: <3 of 5, or pure prose without sections.

### `ARCHITECTURE.md`
**Compatible** if has identifiable sections: tech stack, critical flows or system overview, trade-offs or known issues.
**Partially compatible**: has 2 of 3.
**Incompatible**: <2, or pure decision list (treat as ADR).

### `ROADMAP.md`
**Compatible** if has identifiable phases/cycles/quarters with deliverables and acceptance criteria (or close equivalents).
**Partially compatible**: phases without acceptance criteria.
**Incompatible**: pure backlog list, Trello/CSV export, prose without sequence.

### `PROJECT_STRUCTURE.md`
**Compatible** if has directory tree + conventions section.
**Partially compatible**: directory tree only, or conventions only.
**Incompatible**: prose description without structure.

### `DESIGN_SYSTEM.md`
**Compatible** if has token definitions (color, typography, spacing) + at least 2 of: layout, motion, accessibility, components.
**Partially compatible**: tokens only.
**Incompatible**: aesthetic intent prose without tokens.

### `AGENTS.md`
**Compatible** if has SDI 8-step discipline reference + project context + work tracker.
**Partially compatible**: framework-aware (mentions SDI or has tracker) but missing pieces.
**Incompatible**: not framework-aware, or different agent-rules style.

### `DECISIONS.md` (single file)
**Compatible** if has numbered entries with Context/Decision/Rationale or close.
**Partially compatible**: entries exist but format varies.
**Incompatible**: prose log without entries (rare; usually means ADR-style elsewhere).

### Mapping to strategy

| Compatibility | Default strategy |
|---|---|
| Compatible | A (preserve + header) |
| Partially compatible | B (convert in place — gentle merge) |
| Incompatible | B (convert in place — heavy merge, more user review) or C (rename + generate) — user picks |

## Phase 1.5 decision flow

The skill runs Phase 1.5 **only if** Phase 0 detected one or more existing canonical artifacts. If the project has no existing canonical artifacts, skip Phase 1.5 entirely and go straight to Phase 2.

### Step 1 — Present the matrix

Show the user a table of detected existing artifacts with the proposed default strategy per file:

```markdown
## Existing artifacts detected

| File | Compatibility | Proposed strategy | Why |
|---|---|---|---|
| `README.md` | compatible | **A — preserve + header** | already has stack/summary/doc links |
| `docs/ARCHITECTURE.md` | partially compatible | **B — convert in place** | has stack + flows but no §Trade-offs |
| `docs/ROADMAP.md` | incompatible | **C — rename + generate** | pure Trello export; no phase structure |
| `docs/decisions/` (ADR folder, 12 records) | n/a (special case) | **ADR Option 1** — keep + DECISIONS.md as index | preserves ADR investment |
| `CHANGELOG.md` | n/a | **A — preserve always** | release log |
| `docs/runbooks/` (3 files) | n/a (custom docs) | **A — preserve, link from AGENTS doc map** | |

Confirm or override per file? (per-file response, e.g. "ARCHITECTURE: B, ROADMAP: keep as is = A")
```

### Step 2 — User responds

Accept any of:
- "go" / "ok" / "tudo confirmado" → apply all defaults
- Per-file override: "ROADMAP: A" → user wants to preserve as-is even though incompatible
- "skip [file]: D" → user moves it to external (provides link)
- Mixed responses are fine

### Step 3 — Confirm and proceed

Echo back the final per-file plan, then proceed to Phase 2 with those strategies applied.

If the user says "skip" without specifying handling, **default to A (preserve)**. Better safe.

## Strategy execution rules

When generating in Phase 2, apply the chosen strategy:

### For Strategy A (preserve + header)

1. Read existing file.
2. Prepend the framework header.
3. Write back. Don't touch any other content.

### For Strategy B (convert in place)

1. Read existing file.
2. Parse identifiable sections.
3. Build a new version using the canonical framework template, populating sections from existing content.
4. Anything in existing that doesn't fit canonical sections → place in a `## Legacy notes` section at the end.
5. Add confidence flags per section as appropriate (per `confidence-flags.md`).
6. **Show the diff to the user before writing.**
7. After approval, write the new version.

### For Strategy C (rename + generate)

1. Rename existing: `git mv docs/ROADMAP.md docs/ROADMAP.legacy.md` (or filesystem rename if not git).
2. Generate the canonical fresh version per `artifact-generation-rules.md`.
3. Add note at top of new file: *"Predecessor preserved at [`ROADMAP.legacy.md`](./ROADMAP.legacy.md). Reference for historical context; not the source of truth."*
4. Mention the legacy file in `docs/memory/YYYY-MM-DD.md`.

### For Strategy D (skip)

1. Do NOT generate the artifact.
2. In AGENTS.md, ensure the "External artifact references" section exists and includes this artifact with the user-provided link.
3. In README.md doc map, replace the artifact link with: *"[Artifact name] — external (see AGENTS.md §External artifact references)"*

## Don't-do list

- ❌ Don't silently overwrite ANY existing canonical artifact, even if "incompatible". User picks the strategy.
- ❌ Don't merge content from existing into new without showing the diff. The user must see what changed.
- ❌ Don't reorganize `docs/` folder. Custom content stays where it is.
- ❌ Don't delete `<name>.legacy.md` files later "for cleanup". They're explicitly preserved.
- ❌ Don't try to migrate Notion/Linear/Figma content into the repo. Strategy D exists for a reason.
- ❌ Don't apply ADR Option 2 (convert all ADRs to DECISIONS.md format) unless user explicitly asks. Default is always Option 1 (keep ADRs + index).

## Recovery if convert-to-sdi was already run badly

If a previous convert-to-sdi run already overwrote things (because the older version of this skill didn't have this protocol), the user can recover via git:

1. `git log --diff-filter=M -- docs/PRD.md` to find pre-convert-to-sdi version.
2. Restore the pre-existing version: `git show HEAD~N:docs/PRD.md > docs/PRD.md.recovered`.
3. Re-run convert-to-sdi with the corrected handling.
4. The recovery is logged in `docs/memory/YYYY-MM-DD.md` for traceability.

This is the framework's "we don't lose user content" guarantee. If anything goes wrong, git is the safety net.

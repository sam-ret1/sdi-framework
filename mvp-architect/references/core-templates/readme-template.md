# README Template

The README is the index of the docs bundle. It's the first file the coding agent reads. Keep it tight: ~30–80 lines. No long-form content — link out to the other artifacts.

## Structure

```markdown
# [Product Name]

> One-paragraph product summary. What it does, who it's for, the core value proposition. Read this first; everything else is detail.

## Stack at a glance

| Layer | Choice |
|---|---|
| [Layer e.g. Language] | [Choice e.g. TypeScript] |
| [Layer e.g. Framework] | [Choice] |
| [Layer e.g. Persistence] | [Choice or "n/a"] |
| [Layer e.g. Deployment] | [Choice] |
| ... | ... |

Full rationale lives in [`ARCHITECTURE.md`](./ARCHITECTURE.md).

## Project type

**Type:** [Web SaaS multi-tenant / Landing page / Dashboard / Web API / Mobile app / Data pipeline / AI agent / Automation workflow / Other — describe]
**AI/LLM modifier:** [yes / no]

## Documents

| Doc | Purpose |
|---|---|
| [PRD.md](./PRD.md) | What's being built and why. Users, roles, features in/out of scope. |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Stack, type-specific patterns, critical flows, security, trade-offs. |
| [ROADMAP.md](./ROADMAP.md) | Phased delivery plan with acceptance criteria. |
| [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) | Repo layout and coding conventions. |
| [DESIGN_SYSTEM.md](./DESIGN_SYSTEM.md) | Visual language and tokens. *(only if UI exists)* |
| [IMPLEMENTATION_PLAN_PHASE_1.md](./IMPLEMENTATION_PLAN_PHASE_1.md) *or `IMPLEMENTATION_PLAN_<slug>.md`* | Detailed spec for the current work item. `PHASE_N` for discrete phases; `<slug>` for free-form work (features, maintenance) in ongoing projects. |
| [DECISIONS.md](./DECISIONS.md) | Running paper trail of non-obvious choices. *(populated during implementation)* |
| [AGENTS.md](../AGENTS.md) | Operating discipline for the coding agent (lives at repo root). |

## How to use with a coding agent

Two paths depending on the tool:

**Path A — Claude Code / Codex.** Copy `AGENTS.md` to the repo root. The discipline (audit-first, stop-and-review, DECISIONS.md, round reports) loads automatically. Then paste the kickoff prompt from the current `IMPLEMENTATION_PLAN_*.md` to start the work item.

**Path B — Roo Code / Kilo Code / OpenCode.** Configure the `sdi-mode` custom mode following the appropriate adapter under `sdi-mode/adapters/{tool}.md`. Once active, paste the kickoff prompt from the current `IMPLEMENTATION_PLAN_*.md` to start the work item.

## Open questions deferred to production

[Things that are known but intentionally not decided yet — will be revisited at a specific signal. Examples: payment processor (decide when first paying customer), search engine (decide when corpus exceeds 10k docs), real-time vs polling (decide based on usage patterns at month 2).]

- [Open question — when to decide]
- ...
```

## Writing tips

- **The product summary is one paragraph.** If you need three, you don't understand the product yet.
- **Stack table is compact.** Full rationale belongs in ARCHITECTURE — don't duplicate.
- **Open questions are a feature, not a bug.** They make explicit what is decided to leave undecided. Without them, the next reader assumes everything is final.
- **Do not write the README first.** It's the last artifact, generated after PRD, ARCHITECTURE, ROADMAP, PROJECT_STRUCTURE, and the first IMPLEMENTATION_PLAN exist — it indexes them.

## Common failure modes

- **README full of content instead of links.** README is the menu, not the meal. Link out.
- **Broken links to non-existent docs.** If the project skips DESIGN_SYSTEM (no UI), don't link it.
- **Vague summary.** "A platform for managing things" tells a reader nothing. The summary should differentiate this product from generic alternatives in one sentence.

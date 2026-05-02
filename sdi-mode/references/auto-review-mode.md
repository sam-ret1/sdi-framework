# Auto-Review Mode (default for Checkpoints 2/3/4)

A workflow extension that delegates checkpoint verification to a **reviewer ensemble** — a different-model Opus subagent and a Codex CLI process — escalating to the user only when something needs human judgment. **Default-on** for Checkpoints 2, 3, and 4. The user can opt out per session.

## What this is

Implementation rounds end with structured verdicts from independent reviewers:

- **Attempt 1** runs **two reviewers in parallel**: an Opus subagent (via Anthropic Agent tool) and a Codex CLI process (`codex exec`, typically gpt-5.5 with reasoning effort `xhigh` per the user's `~/.codex/config.toml`).
- **Attempts 2 and 3** (if needed after a FAIL) run the retry reviewer: Opus subagent by default. If the runtime cannot provide an Opus subagent but Codex produced a usable attempt-1 review, use Codex as the degraded retry reviewer and note that in the round report. If no retry reviewer is available, escalate to the user.

The ensemble on attempt 1 is load-bearing: empirically, Opus and Codex find partially-disjoint bugs (in this framework's validation runs: Codex 5/5 planted + 6 unplanted; Opus 4/5 planted + 3 unplanted; the union catches more than either alone). For retries, the bug surface has shrunk to verifying the fix, so one retry reviewer is sufficient and saves cost/latency as long as the prior findings are included in the retry packet.

Both reviewers receive the **same self-contained prompt** (the adversarial review template below) with explicit cross-file checks (CSS classes used vs defined, API contracts caller-vs-handler, report-vs-reality, optimistic UI revert paths). Outputs are captured to `docs/reviews/round-XN-attempt-N-{opus,codex}.md` for audit trail.

Goals:
- Remove friction at low-stakes checkpoints without losing review depth.
- Get model diversity exactly where it matters most (first review of fresh code).
- Keep the discipline binary (gates pass or fail; no fuzzy "looks good").
- Preserve full audit trail so the user can spot-check.
- Escalate quickly when something needs human judgment.

Non-goals:
- Replacing human review of decisions. Anything that needs a `DECISIONS.md` entry escalates.
- Speeding up the implementer agent itself (auto-review is purely about the gate at the end of each round).
- Working without explicit per-gate criteria. A vague "review this" prompt produces theater.

## Per-round commit convention

Auto-review is structured around per-round git commits.

- **At the start of each round**, the implementer captures `BASE_SHA = git rev-parse HEAD` (the last commit of the previous round, or the phase-start commit for round A) and records it in the round report header.
- **At the end of each round**, the implementer commits with the message format `round X/CN: <summary>` (e.g. `round B/C2: callback + middleware`).
- **Fix attempts** triggered by an auto-review FAIL get their own commits with the format `round X/CN fix N: <what>` (e.g. `round B/C2 fix 1: thread returnTo through Server Action`).
- **Review artifacts** are committed only after the auto-review loop closes (PASS, ESCALATE, or loop cap). Before invoking reviewers, write the round report draft to `docs/reviews/round-XN-report.md`. Keep that report and `docs/reviews/round-XN-attempt-N-{reviewer}.md` uncommitted while attempts are still running. After closure, commit the report + reviewer outputs together with `round X/CN review artifacts: <verdict>`.
- **No squashing.** The granular commit history IS the audit trail. The user does not need to clean up before merging.

The reviewers read the code diff via `git diff BASE_SHA..HEAD` — so BASE_SHA stays fixed across all attempts in a round. The report draft is read directly from `[ROUND_REPORT_PATH]`; it does not need to be committed for reviewers to inspect it. Each attempt re-evaluates the round's deliverable in full (initial commit + any fix commits), not just the last fix delta. This protects against fix-attempt-N reintroducing a problem that passed in fix-attempt-N-1.

## Disabling and re-enabling per session

**Auto-review is the default. Every new session starts with auto-review on.**

**Opt-out per session:** the user says any of:
- "user-review for this phase"
- "review the next round myself"
- "stop auto-reviewing"
- "back to user-gated"
- "no auto-review"

When opted-out, the implementer drops back to user-gated for the rest of the session — every checkpoint waits for an explicit user "go".

**Re-enable per session:** the user says any of:
- "auto-review again"
- "go back to auto-review"
- "switch back to auto-review"

**Persistence:** none. Every new session starts in default (auto-review on). This is intentional — opt-out is per-phase or per-session work-style, not a project-level setting.

**Where to record it:** if the user opts out, mention in today's `docs/memory/YYYY-MM-DD.md` entry (e.g., "User opted out of auto-review this session for the rest of Phase 2"). Breadcrumb only; the mode itself lives in the conversation.

## When auto-review applies

| Checkpoint | Eligibility |
|---|---|
| 1 — Foundation | **User-gated always.** Audit findings need human resolution; downstream cost of errors is high. |
| 2 — Core domain logic | **Auto-review default-on.** |
| 3 — Wire up integrations | **Auto-review default-on.** |
| 4 — UI | **Auto-review default-on.** |
| 5 — Housekeeping | **User-gated always.** Final acceptance is a human decision. |

Even within a default-on checkpoint, the round escalates immediately when any of the always-escalate triggers below fire.

## Always-escalate triggers (override auto-review default)

If any of these occur during the round, the implementer skips auto-review for that round and stops for the user:

1. **The round produced (or should produce) a `DECISIONS.md` entry.** A material divergence, a deferred feature, a non-obvious trade-off, a deviation from convention — by definition a judgment call. See `decisions-log-format.md` for what qualifies.
2. **Blocker encountered during implementation.** Anything that prevents finishing the round (missing dependency the user needs to install, contradictory plan content, broken external service).
3. **Emergency deviation.** Per `stop-and-review-patterns.md` — security bug, data-loss risk, regression of previously-working functionality.
4. **Schema migration with data-loss risk** (drop column, NOT NULL on existing column without backfill, type change that loses precision). Even if tests pass, the user must approve.
5. **New external dependency added** beyond what the plan listed. Plan said use library X; the implementer pulled in library Y too. Escalate.
6. **Security-relevant change** beyond plan scope (auth helper added/changed, RLS policy modified, secret-handling code touched, CORS or CSP changed).
7. **Plan revision (`rN`) added during the round.** A revision means reality diverged from plan; the user should see what changed before the next round proceeds.
8. **PRD or ARCHITECTURE deviation.** If implementing a §2 surface required deviating from the higher-precedence doc, escalate (see `expected-artifacts.md` precedence rules).

Surface the trigger explicitly when escalating: "Auto-review skipped because [trigger]. Stopping for your review."

## The loop

```
┌────────────────────────────────────────┐
│ Implementer finishes round and commits │
│  ("round X/CN: <summary>")             │
└────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────┐
│ Always-escalate trigger fired?         │──── yes ──► Stop for user
│ OR user opted out this session?        │
└────────────────────────────────────────┘
                 │ no
                 ▼
┌────────────────────────────────────────┐
│ Build review packet                    │
│ (substitute placeholders into prompt   │
│  template; write to                    │
│  .sdi-review-prompt-tmp.txt)           │
└────────────────────────────────────────┘
                 │
                 ▼
        ┌────────┴─────────┐
        │ Attempt number?  │
        └────────┬─────────┘
                 │
        ┌────────┴─────────┐
        ▼                  ▼
   Attempt 1            Attempts 2-3
        │                  │
        ▼                  ▼
┌──────────────────┐  ┌──────────────────┐
│ Run BOTH in      │  │ Run retry        │
│ parallel:        │  │ reviewer         │
│  - Opus subagent │  │ (Opus default;   │
│    (Agent tool)  │  │ degraded Codex   │
│  - codex exec    │  │ only if needed)  │
│ Outputs:         │  │   round-XN-      │
│  docs/reviews/   │  │   attempt-N-     │
│   round-XN-      │  │   opus.md        │
│   attempt-1-     │  │                  │
│   {reviewer}.md  │  │                  │
│  docs/reviews/   │  │                  │
│   round-XN-      │  │                  │
│   attempt-1-     │  │                  │
│   codex.md       │  │                  │
└──────────────────┘  └──────────────────┘
        │                  │
        ▼                  ▼
┌──────────────────┐  ┌──────────────────┐
│ Both ran?        │  │ Run succeeded?   │
└──────────────────┘  └──────────────────┘
        │                  │
   ┌────┴────┐         ┌───┴────┐
   │ both    │         │ yes    │
   ▼         ▼         ▼        ▼
 yes       one fail   ok      no (escalate)
   │         │         │
   ▼         ▼         │
┌──────────────────┐   │
│ Merge verdicts:  │   │
│ - PASS only if   │   │
│   both PASS      │   │
│ - FAIL if either │   │
│   FAIL           │   │
│ - ESCALATE if    │   │
│   either         │   │
│   ESCALATE       │   │
└──────────────────┘   │
        │              │
   one fail (a1)       │
        ▼              │
┌──────────────────┐   │
│ Continue with    │   │
│ surviving        │   │
│ reviewer's       │   │
│ verdict; note    │   │
│ skip in report   │   │
└──────────────────┘   │
        │              │
        ▼              ▼
┌────────────────────────────────────────┐
│ Both reviewers failed (a1)?            │──── yes ──► Stop for user
│ (codex CLI down + Opus subagent error) │           ("Auto-review
└────────────────────────────────────────┘            unavailable")
                 │ no
                 ▼
        ┌────────┴────────┐
        ▼                 ▼
      PASS              FAIL/ESCALATE
        │                 │
        │                 ▼
        │         ┌─────────────────┐
        │         │ ESCALATE?       │── yes ──► Stop for user
        │         └─────────────────┘
        │                 │ no (FAIL)
        │                 ▼
        │         ┌─────────────────┐
        │         │ attempt < 3?    │── no  ──► Stop for user
        │         └─────────────────┘            ("loop cap hit")
        │                 │ yes
        │                 ▼
        │         ┌─────────────────┐
        │         │ Apply union of  │
        │         │ findings, commit│
        │         │ ("round X/CN    │
        │         │   fix N: ...")  │
        │         └─────────────────┘
        │                 │
        │                 └──► back to "Build review packet"
        ▼                       (BASE_SHA unchanged; next attempt = retry reviewer)
┌────────────────────────────────────────┐
│ Append history to round report         │
│ Delete .sdi-review-prompt-tmp.txt      │
│ Propose next round (still stop briefly │
│ to deliver the report; user may        │
│ interject before next round starts)    │
└────────────────────────────────────────┘
```

PASS does not mean "skip the round report" — the implementer still posts the report (with auto-review history attached). It means the user does not have to gate-check explicitly; they can let the next round start, or interject if they want to look closer.

## Verdict merging (attempt 1 only)

When both reviewers ran, merge verdicts mechanically:

| Opus | Codex | Merged |
|---|---|---|
| PASS | PASS | **PASS** |
| PASS | FAIL | FAIL |
| FAIL | PASS | FAIL |
| FAIL | FAIL | FAIL |
| any | ESCALATE | ESCALATE |
| ESCALATE | any | ESCALATE |

Rules in plain language:
- **PASS only when both PASS.** Either reviewer's FAIL or ESCALATE blocks PASS.
- **ESCALATE wins over FAIL.** If either reviewer escalates, the round escalates — the user must judge.
- **Findings unionize.** When the merged verdict is FAIL, fix attempts must address the union of findings from both reviewers, not just one.

## Reviewer fallback (one reviewer fails to run)

If a reviewer cannot be invoked or produces unusable output (binary missing, network down, malformed output, no parseable `VERDICT:` line):

| Situation | Action |
|---|---|
| One reviewer ok, one failed | Continue with the one that ran. Note the skip in the round report ("codex skipped: <reason>" or "opus subagent skipped: <reason>"). Mark the mode as `degraded` in the auto-review history, but do NOT downgrade the surviving reviewer's verdict — if Opus alone returned PASS, treat as PASS. |
| Both reviewers failed | **Escalate to the user.** Surface: "Both reviewers failed: [opus reason], [codex reason]. Auto-review unavailable for this round — please review manually or fix the reviewer setup." |

For attempts 2 and 3, use the configured retry reviewer. Default is Opus. If Opus is unavailable in this runtime and Codex produced the only usable attempt-1 review, Codex may be used as the degraded retry reviewer. If the configured retry reviewer fails, escalate immediately.

## Building the review packet

Before invoking either reviewer, build a self-contained prompt by substituting placeholders into the template in §"Adversarial review prompt template" below. Substitute BEFORE writing the prompt to disk / passing to the subagent — neither reviewer expands placeholders.

| Placeholder | Source |
|---|---|
| `[ROUND_ID]` | e.g., "Phase 2 — Round B — Checkpoint 2 Core" |
| `[STACK]` | one-line stack summary from `AGENTS.md` (e.g., "Next.js 15 App Router + Drizzle + Postgres + Supabase Auth") |
| `[BASE_SHA]` | the SHA captured at the start of this round (recorded in the round report header) |
| `[ROUND_REPORT_PATH]` | path to the round report draft/final artifact, e.g. `docs/reviews/round-B-C2-report.md` |
| `[PLAN_PATH]` | path to the implementation plan, e.g. `docs/IMPLEMENTATION_PLAN_PHASE_2.md` |
| `[PLAN_SECTIONS]` | the §s relevant to this round, e.g. "§3, §6.1, §11" |
| `[PRIOR_REVIEW_FINDINGS]` | attempt 1+ findings that must be re-checked on retries; use "None — first attempt." for attempt 1 |

Write/update the round report draft at `[ROUND_REPORT_PATH]` before invoking reviewers. Then write the substituted prompt to `.sdi-review-prompt-tmp.txt` at the repo root (used as stdin to codex; the same file content is passed as the `prompt` parameter to the Opus subagent). Add this file to `.gitignore` if it isn't already (it is temporary and recreated per attempt). Delete after the round closes.

The same packet goes to both reviewers — they see identical input. Outputs are captured per reviewer:
- Opus subagent: returned as the Agent tool's text result; the implementer writes it to `docs/reviews/round-XN-attempt-N-opus.md`.
- Codex: written directly via `--output-last-message docs/reviews/round-XN-attempt-N-codex.md`.

These output files ARE committed for audit trail after the auto-review loop closes, together with the final round report.

Do **not** include in the packet:
- Prior conversation context.
- Prior rounds' diffs (only this round, computed by each reviewer via `git diff [BASE_SHA]..HEAD`).
- Memory entries or DECISIONS log directly (each reviewer reads them itself by grepping the repo when needed; the packet stays focused on this round's diff and report).

Exception: retry packets MUST include prior auto-review findings in `[PRIOR_REVIEW_FINDINGS]` so the retry reviewer verifies fixes for both Opus and Codex findings from earlier attempts.

## Adversarial review prompt template

The template below is the prompt sent to reviewers. Copy verbatim, then replace `[ROUND_ID]`, `[STACK]`, `[BASE_SHA]`, `[ROUND_REPORT_PATH]`, `[PLAN_PATH]`, `[PLAN_SECTIONS]`, and `[PRIOR_REVIEW_FINDINGS]`.

```
You are an adversarial code reviewer for round [ROUND_ID] of a [STACK] implementation.

Workdir: project root.
The implementation is committed at HEAD; the previous round (baseline) is at commit [BASE_SHA].
The implementer wrote a round report at [ROUND_REPORT_PATH] claiming what was built and what gates pass.
The plan section relevant to this round is [PLAN_PATH] §[PLAN_SECTIONS].
Prior review findings that MUST be verified this attempt: [PRIOR_REVIEW_FINDINGS]

Your job: find what is broken AND what the report misrepresents. Trust only what you can verify; the implementer is not adversarial but wants to ship, so they may overstate.

## Steps you must perform

1. Run `git diff [BASE_SHA]..HEAD` to see the full round diff.
2. Read [ROUND_REPORT_PATH] end to end.
3. Read [PLAN_PATH] sections [PLAN_SECTIONS] to know what this round was supposed to deliver.
4. Read AGENTS.md for stack and conventions.
5. For each concrete claim in the round report, verify it against the diff and the actual file system. Use git/ls/cat/grep as needed.
6. If prior review findings are listed, verify each one was actually fixed. Do not PASS a retry until every prior finding is either fixed or explicitly escalated.

## Things you MUST actively check (beyond standard code review)

A. Report-vs-reality for files. For each NEW or MODIFIED file in the round report, verify it exists in the diff. Fabricated file claims are bugs.
B. Report-vs-reality for tests. For each test claim ("N tests pass"), verify the test file exists and contains tests matching the claim. No test file = fabricated evidence.
C. CSS class definitions. For every className referenced in JSX/TSX, verify the class is defined in styles/globals.css, a CSS module, tailwind.config.*, or is a built-in tailwind utility. Undefined classes are bugs.
D. API contracts. For every fetch/POST/PUT/PATCH in client code, locate the route handler and compare request body shape vs handler validation (Zod schema). Mismatches are bugs.
E. Optimistic UI patterns. For setState before await fetch, verify error branch reverts state. Logging only is a bug.
F. Plan-vs-implementation. For each gate marked ✓ in the report, locate the evidence in the diff. Missing evidence = flag.
G. Report wording precision. UI behavior phrasing in the report must match code exactly.
H. Stack-specific architecture. Adapt to [STACK]: in-memory state in serverless, sync APIs in RSC, missing root layouts, RLS bypass, race conditions, unhandled promise rejections.
I. DECISIONS log. For non-obvious choices in the diff, grep DECISIONS.md for an entry. Unflagged choices ESCALATE.

## Bug classes

1. Internal inconsistency — report claims X, code does Y; or two parts of the code disagree.
2. Contract mismatch — caller sends one shape, handler expects another.
3. Missing prerequisite — code or report references a file, component, helper, hook, env var, test, or convention that doesn't exist.
4. Vague or unverifiable claim — gate or report claim that cannot be marked ✓/✗ from evidence.
5. DECISIONS-worthy choice without flag.
6. Convention or architecture mistake.
7. Anything else surprising or risky.

## Output format

Findings-first. Do not summarize what works. Each finding:
- Class (1–7).
- Where (file:line).
- What is wrong (one sentence).
- Why it bites (one sentence).
- Suggested fix (one sentence).

End with TWO lines:
- TOTAL FINDINGS: N. By class: 1=a, 2=b, 3=c, 4=d, 5=e, 6=f, 7=g.
- VERDICT: PASS / FAIL / ESCALATE.

VERDICT rules:
- PASS = zero findings of class 1–6.
- FAIL = at least one finding of class 1–4 or 6 (mechanically fixable).
- ESCALATE = at least one finding of class 5, OR class 7 marked urgent, OR anything requiring user judgment.

Do NOT edit any files. Your response is the review report itself.
```

## Verdict format (what the implementer expects back)

Each reviewer returns its review as Markdown text. The implementer parses the last `VERDICT:` line of each:

- **PASS** — all gates verified ✓ with citations; no escalation triggers found.
- **FAIL** — one or more gates ✗, but the failures are fixable mechanically (e.g., contract mismatch, missing test for an edge case the plan listed, observability hook not wired, undefined CSS class).
- **ESCALATE** — one or more findings require user judgment (DECISIONS-worthy choice without flag, urgent class-7 finding, anything outside the implementer's scope).

If the verdict line is missing, malformed, or the output is empty/garbage, treat that reviewer as failed (see §"Reviewer fallback"). Do not reject a short but well-formed PASS response solely because it is brief.

After parsing, apply the merge table above (attempt 1 only) and proceed per the loop.

## Loop cap

- **Maximum 3 review attempts per round** (1 ensemble attempt + 2 single-reviewer retries).
- After attempt 3 still FAIL → stop and escalate to the user with the full review history.
- Hitting the cap is a signal that something structural is wrong — either the gate criteria are not the right ones for this round, or the round is doing more than it should, or there's a bug the agents can't see. The implementer says so explicitly when surfacing: "Auto-review hit 3 attempts without PASS — escalating; this usually means a structural issue worth a closer look."

## Auto-review history in round report

Every auto-reviewed round must include the auto-review history in its round report. See `round-report-template.md` for the exact section shape.

For each attempt:
- Attempt N — reviewers run (`opus + codex` on attempt 1; Opus retry reviewer on attempts 2-3 unless the runtime is in a documented degraded reviewer mode) and verdict (merged on attempt 1, direct on retries).
- Per reviewer: full structured output verbatim from `docs/reviews/round-XN-attempt-N-{reviewer}.md` (findings + verdict line).
- Issues found (if any) and the fix applied between attempts (with the fix commit SHA).

Do not summarize ("3 attempts, eventual PASS"). The user uses the history to spot-check the reviewers' calls — omitting it defeats the purpose of the audit trail.

## Invocation — Opus subagent (Agent tool)

When the host runtime supports it, the Opus subagent runs via the Anthropic Agent tool. Use the `general-purpose` subagent type with explicit `model: "opus"` so the subagent runs Opus regardless of the parent session's model. If the host runtime has no Agent tool or no Opus model selection, record "opus subagent unavailable: <reason>" and apply reviewer fallback.

Inputs:
- `description`: short, e.g. `Round B/C2 attempt 1 review`.
- `subagent_type`: `general-purpose`.
- `model`: `opus`.
- `prompt`: the substituted contents of `.sdi-review-prompt-tmp.txt` (the entire adversarial review prompt with placeholders filled in).

The subagent inherits file/Bash tools so it can run `git diff`, read files, and grep the repo as the prompt instructs. The text response is the review report — write it to `docs/reviews/round-XN-attempt-N-opus.md` for audit trail.

If the parent session is already running Opus, the subagent still runs as a separate process with no shared conversation context — that's the load-bearing piece (independent reasoning over the same inputs).

## Invocation — Codex CLI (codex exec)

```
codex exec --ephemeral \
  --sandbox read-only \
  --output-last-message docs/reviews/round-XN-attempt-N-codex.md \
  -C [REPO_ROOT] \
  - < .sdi-review-prompt-tmp.txt
```

Notes:
- `--ephemeral` — codex doesn't persist this run as a session.
- `--sandbox read-only` — reviewer runs read-only; the CLI still writes the final message to `--output-last-message`.
- `--output-last-message <FILE>` — captures only the final agent message in the output file. Skips intermediate stdout noise (e.g., PowerShell profile errors on Windows machines that have script execution disabled).
- `-C [REPO_ROOT]` — sets codex's working directory.
- `- < <prompt-file>` — passes the prompt via stdin (cleaner than escaping inline arguments).

Windows note: if PowerShell resolves `codex` to an npm `codex.ps1` shim and script execution is disabled, invoke from a real Bash shell as shown above, or call the `codex.exe` binary directly in the tool-specific command wrapper.

The run typically takes 3–10 minutes with xhigh reasoning on a meaningful round diff. Run as a background command (your tool's mechanism for long-running shell commands) and let the user know you're waiting.

## Parallel orchestration (attempt 1 only)

On attempt 1, the implementer fires both reviewers and waits for both:

1. Write/update `docs/reviews/round-XN-report.md` as the report draft for this attempt.
2. Write the substituted prompt to `.sdi-review-prompt-tmp.txt` with `[PRIOR_REVIEW_FINDINGS] = "None — first attempt."`.
3. Spawn the Opus subagent (Agent tool, `run_in_background: true`).
4. Start `codex exec` as a background Bash command.
5. Wait for both to complete (the runtime notifies on each completion).
6. Read both output files. Parse VERDICT from each. Apply the merge table.
7. If either reviewer failed to produce usable output, apply §"Reviewer fallback".

On attempts 2 and 3, update the report draft and substitute `[PRIOR_REVIEW_FINDINGS]` with the union of findings from earlier attempts plus the fix commit(s) that claim to address them. Fire the configured retry reviewer (Opus by default; degraded Codex-only only when Opus is unavailable and Codex was the surviving attempt-1 reviewer). Wait, parse VERDICT, proceed per the loop.

After the round closes (PASS or escalation), append final auto-review history to `docs/reviews/round-XN-report.md`, delete `.sdi-review-prompt-tmp.txt` (it's gitignored and recreated per attempt anyway), and commit the round report + per-attempt output files with `round X/CN review artifacts: <verdict>`. Do NOT delete the per-attempt output files in `docs/reviews/` — those are the audit trail.

## Common pitfalls

- **Vague gate criteria.** "Verify the integration tests pass" is not enough. The gate must say what counts as ✓ ("Integration test count matches plan §8 list and all pass per the runner output below"). Without that, even adversarial prompts will rubber-stamp.
- **Forgetting to substitute placeholders.** Neither reviewer expands `[BASE_SHA]` or `[ROUND_REPORT_PATH]` — substitute them into the prompt template before writing to disk / passing to the subagent, or the run is wasted.
- **Each reviewer is a fresh subprocess with no shared context.** Build the packet to be entirely self-contained. The parent session's conversation context is invisible to both reviewers.
- **If the diff is too large for either reviewer's context, the round was too big.** Modern models handle 100k+ tokens, but extremely large rounds may overflow. Split the round (one commit's worth of work each) and re-invoke per chunk.
- **Treating PASS as "skip the round report".** PASS still requires the report. The user reads the report to track progress; the auto-review history is an addendum, not a replacement.
- **Looping on cosmetic issues.** If attempt 2 fixed the substantive gate failure but attempt 3 fails on a tiny new issue, the loop cap is your friend — escalate, don't go for attempt 4.
- **Ignoring escalation triggers.** An always-escalate trigger means stop, period. Don't try to pre-resolve it and skip to auto-review; the user must see it.
- **Letting reviewers edit code.** The review prompt instructs read-only. If a gate failure requires a fix, the implementer makes the fix between attempts.
- **Codex CLI assumptions.** The framework assumes `codex` is on PATH and the user has `~/.codex/config.toml` (or `$CODEX_HOME/config.toml`) selecting an appropriate reviewer model. The framework does not configure this. If `codex --version` fails, surface the gap to the user — attempt 1 falls back to Opus-only per §"Reviewer fallback" (note in the round report). On Windows, PowerShell may resolve `codex` to a blocked npm `.ps1` shim; use Bash or call `codex.exe` directly.
- **Opus subagent assumptions.** Full ensemble mode assumes the host runtime supports the Anthropic Agent tool with model selection. On runtimes without that, Opus-as-subagent is unavailable: attempt 1 may run Codex-only in degraded mode, and retries may use Codex as the retry reviewer only if Codex produced the surviving attempt-1 review. If no independent reviewer can run, escalate or opt out of auto-review.
- **`codex review --base/--commit` parser quirk.** The CLI rejects custom `[PROMPT]` when `--base` or `--commit` is set. This protocol uses `codex exec` (not `codex review`) precisely to bypass that limitation. Do not switch to `codex review`.
- **Sequencing the reviewers serially.** On attempt 1, run both in parallel, not one after the other. Serial costs ~2x the wall-clock latency for no benefit. The runtime supports parallel Agent + Bash background invocations.

## When auto-review is wrong

Sometimes auto-review is the wrong default for a phase even though it's enabled by default:

- **High-uncertainty work.** New domain, new stack, exploratory architecture — keep user-gated. The cost of an auto-pass on a poorly-understood checkpoint exceeds the friction savings.
- **Shipping a release candidate.** Higher stakes; extra human review is worth the friction.
- **After a series of FAILs.** If multiple recent rounds in this phase needed retries, drop back to user-gated until the issue is understood.

The user can opt out at any point. The implementer should also propose dropping back to user-gated when it sees these patterns: "Three recent rounds needed retries — recommend opting out of auto-review for the rest of this phase. OK?"

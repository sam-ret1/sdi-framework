# Auto-Review Mode (opt-in)

A workflow extension that delegates checkpoint verification to a fresh subagent for low-stakes checkpoints, escalating to the user only when something needs human judgment. **Default is off** — the user enables it explicitly, session-scoped.

## What this is

In default sdi-mode, every checkpoint stops and waits for explicit user go (see `stop-and-review-patterns.md`). For projects in active development this can be too much friction at mid-phase rounds where the discipline is mechanical (run gate checklist, confirm tests pass, confirm no surprises).

Auto-review replaces the user-go gate at **Checkpoints 2, 3, and 4** with a structured subagent verdict. Checkpoints 1 (Foundation) and 5 (Housekeeping), and any round that hits an always-escalate condition, stay user-gated regardless.

Goals:
- Remove friction at low-stakes checkpoints.
- Keep the discipline binary (gates pass or fail; no fuzzy "looks good").
- Preserve full audit trail so the user can spot-check.
- Escalate quickly when something needs human judgment.

Non-goals:
- Replacing human review of decisions. Anything that needs a `DECISIONS.md` entry escalates.
- Speeding up the implementer agent itself (auto-review is purely about the gate at the end of each round).
- Working without explicit per-gate criteria. A vague "review this" prompt produces theater.

## Enabling and disabling

**Enable per session:** the user says something like "use auto-review for this phase", "switch to auto-review", "go auto for the next checkpoint". The implementer applies it for the rest of the session unless told otherwise.

**Disable per session:** the user says "back to user-review", "stop auto-reviewing", "I want to review the next round myself". Auto-review also auto-disables on session end — it never persists across sessions.

**Persistence:** none. Each new session starts in default (user-gated) mode. This is intentional — auto-review is opt-in per phase of work, not a project-level setting.

**Where to record it:** mention in today's `docs/memory/YYYY-MM-DD.md` entry (e.g., "Auto-review enabled this session for Checkpoints 2–4"). This is a breadcrumb only; the mode itself lives in the conversation.

## When auto-review applies

| Checkpoint | Eligibility |
|---|---|
| 1 — Foundation | **User-gated always.** Audit findings need human resolution; downstream cost of errors is high. |
| 2 — Core domain logic | **Auto-review eligible.** |
| 3 — Wire up integrations | **Auto-review eligible.** |
| 4 — UI | **Auto-review eligible.** |
| 5 — Housekeeping | **User-gated always.** Final acceptance is a human decision. |

Even within an eligible checkpoint, the round escalates immediately when any of the always-escalate triggers below fire.

## Always-escalate triggers (override auto-review eligibility)

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
│ Implementer finishes round             │
└────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────┐
│ Always-escalate trigger fired?         │──── yes ──► Stop for user
└────────────────────────────────────────┘
                 │ no
                 ▼
┌────────────────────────────────────────┐
│ Build review packet                    │
│ (diff + plan §s + gates + criteria)    │
└────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────┐
│ Invoke subagent with packet            │  attempt N (max 3)
└────────────────────────────────────────┘
                 │
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
        │         │ Fix issues      │
        │         └─────────────────┘
        │                 │
        │                 └──► back to "Build review packet"
        ▼
┌────────────────────────────────────────┐
│ Append history to round report         │
│ Propose next round (still stop briefly │
│ to deliver the report; user may        │
│ interject before next round starts)    │
└────────────────────────────────────────┘
```

PASS does not mean "skip the round report" — the implementer still posts the report (with auto-review history attached). It means the user does not have to gate-check explicitly; they can let the next round start, or interject if they want to look closer.

## Building the review packet

The packet is a self-contained prompt for a fresh subagent. The subagent does not see the prior conversation. Include:

1. **Round identifier** — phase/work-item name, round letter/number, target checkpoint (e.g., "Phase 2 — Round B — Checkpoint 3 Wire up integrations").
2. **Diff** — full unified diff of files changed in this round. If the round touched many files, summarize unchanged ones; never omit changed ones.
3. **Plan §s touched** — the relevant subsections of `IMPLEMENTATION_PLAN_*.md` for this checkpoint (per §11 of the plan). Pull the actual text of those §s into the packet — the subagent does not have access to read the plan unless you give it the content.
4. **Canonical gate checklist for this checkpoint** — copy the exact gate items from `stop-and-review-patterns.md` for the target checkpoint.
5. **Phase-specific gate additions** — copy any phase-specific gates from §11 of the plan for this checkpoint.
6. **Per-gate verifiable criteria** — for each gate, an explicit "how to verify" statement. See the prompt template below.
7. **Test runner output** — paste the actual test results (counts, pass/fail). The subagent should not re-run tests; it verifies counts against the plan.
8. **Round report draft** — the implementer's draft round report with all sections except the "Auto-review history" section.

Do **not** include:
- Prior conversation context.
- Prior rounds' diffs (only the current round).
- Memory entries or DECISIONS log (the subagent should not be deciding; if a DECISIONS-worthy item exists, the round should have escalated).

## Subagent prompt template

```markdown
You are a checkpoint reviewer. You verify that a round of implementation work satisfies a binary gate checklist. You return a structured PASS / FAIL / ESCALATE verdict with file:line evidence per gate.

## Hard rules

1. **Cite file:line for every ✓.** If you cannot cite specific evidence in the diff or plan content, mark the gate ✗ — never ✓ without evidence.
2. **Do not rubber-stamp.** "Looks fine" is not a valid verdict. Each gate is binary; either you can prove it from the diff and the test output, or you cannot.
3. **Escalate non-obvious choices.** If you find that the implementer made a judgment call (chose one library over another, deferred a feature, deviated from convention, accepted a trade-off), return ESCALATE — that's a `DECISIONS.md`-worthy item the user must see.
4. **Do not ask follow-up questions.** Return one structured verdict; the implementer cannot answer you.
5. **Do not re-run tests.** Verify the test counts in the round report against the plan's test list. If they don't match, mark ✗.
6. **Do not edit code.** You are read-only.

## What you receive

- Round identifier (phase/work-item, round, checkpoint).
- Diff of files changed in this round.
- Relevant §s of the implementation plan.
- Canonical gate checklist for this checkpoint (from `stop-and-review-patterns.md`).
- Phase-specific gate additions (from §11 of the plan).
- Per-gate verification criteria.
- Test runner output.
- The implementer's round report draft.

## What you return

Use this exact structure:

## Auto-review — [round identifier]

### Per-gate verdicts

For each gate (canonical + phase-specific), one line:
- [✓ or ✗] **Gate**: "[gate text]"
  - **Evidence** (if ✓): `path/to/file.ext:LINE` — [one short sentence describing what the citation shows]
  - **Failure** (if ✗): [one short sentence describing what's missing or wrong]

### Verdict

One of:
- **PASS** — all gates ✓, no escalation triggers found.
- **FAIL** — one or more gates ✗, all of which are fixable in a follow-up iteration (not DECISIONS-worthy, not blocker-class).
- **ESCALATE** — one or more findings require human judgment (DECISIONS-worthy choice, blocker, emergency, or out-of-scope deviation).

### Issues (only if FAIL or ESCALATE)

For each issue:
1. **[fixable | escalation]** — [short title]
   - What: [one sentence]
   - Where: `path/file.ext:LINE` (if applicable)
   - Recommendation: [one sentence — fix or escalate]

### Notes

[Optional — anything not covered by the above that the user should know if they spot-check this review later. Keep to ≤3 lines.]
```

The subagent's output is machine-readable in shape but human-readable in content. The implementer pastes it verbatim into the round report.

## Verdict format (what the implementer expects back)

The subagent must return one of three verdicts:

- **PASS** — all gates verified ✓ with citations; no escalation triggers found. Implementer continues to the next round.
- **FAIL** — one or more gates ✗, but the failures are fixable mechanically (e.g., missing test for an edge case the plan listed; observability hook not wired; helper not exported). Implementer fixes and re-invokes review.
- **ESCALATE** — one or more findings require user judgment. Implementer stops and surfaces to the user.

If the verdict is ambiguous or the subagent ignored the structure (returned prose, "looks good", etc.), treat it as ESCALATE — the discipline's value depends on structure.

## Loop cap

- **Maximum 3 review attempts per round** (1 initial + 2 retries).
- After attempt 3 still FAIL → stop and escalate to the user with the full review history.
- Hitting the cap is a signal that something structural is wrong — either the gate criteria are not the right ones for this round, or the round is doing more than it should, or there's a bug the agents can't see. The implementer says so explicitly when surfacing: "Auto-review hit 3 attempts without PASS — escalating; this usually means a structural issue worth a closer look."

## Auto-review history in round report

Every auto-reviewed round must include the auto-review history in its round report. See `round-report-template.md` for the exact section shape.

For each attempt:
- Attempt N — verdict (PASS / FAIL / ESCALATE).
- Per-gate ✓/✗ with file:line evidence.
- Issues found (if any) and the fix applied between attempts.

Do not summarize ("3 attempts, eventual PASS"). The user uses the history to spot-check the subagent's calls — omitting it defeats the purpose of the audit trail.

## Common pitfalls

- **Vague gate criteria.** "Verify the integration tests pass" is not enough. The gate must say what counts as ✓ ("Integration test count matches plan §8 list and all pass per the runner output below"). Without that, the subagent will rubber-stamp.
- **Sending the full conversation context.** The Agent tool starts a fresh subagent. The packet must be self-contained. Sending more does not help — it dilutes the verifiable criteria.
- **Treating PASS as "skip the round report".** PASS still requires the report. The user reads the report to track progress; the auto-review history is an addendum, not a replacement.
- **Looping on cosmetic issues.** If attempt 2 fixed the substantive gate failure but attempt 3 fails on a tiny new issue, the loop cap is your friend — escalate, don't go for attempt 4.
- **Ignoring escalation triggers.** An always-escalate trigger means stop, period. Don't try to pre-resolve it and skip to auto-review; the user must see it.
- **Letting the subagent edit code.** The subagent reads, doesn't write. If a gate failure requires a fix, the implementer makes the fix between attempts.

## Sample invocation (Claude Code, Agent tool)

```
Agent({
  subagent_type: "general-purpose",
  description: "Review Round B Checkpoint 3",
  prompt: <<full review packet built per the §"Building the review packet" rules>>
})
```

In Codex / Roo / Kilo / OpenCode, use the equivalent delegation primitive. The discipline is the same; only the mechanics differ.

## When auto-review is wrong

Sometimes auto-review is the wrong default for a phase even though it's enabled:

- **High-uncertainty work.** New domain, new stack, exploratory architecture — keep user-gated. The cost of an auto-pass on a poorly-understood checkpoint exceeds the friction savings.
- **Shipping a release candidate.** Higher stakes; extra human review is worth the friction.
- **After a series of FAILs.** If multiple recent rounds in this phase needed retries, drop back to user-gated until the issue is understood.

The user can disable auto-review at any point. The implementer should also propose dropping back to user-gated when it sees these patterns: "Three recent rounds needed retries — recommend disabling auto-review for the rest of this phase. OK?"

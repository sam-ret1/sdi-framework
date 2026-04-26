# Round Report Template

Every implementation round ends with a report. Same sections, same order. Users read many of these; consistency lets them scan quickly.

## The shape

```markdown
## Round [letter or number] — [one-line description]

### Delivered

[Table or bullet list. For each file touched or created, one line:
 - Path
 - What it does / why it changed
 - Test coverage (counts or "n/a")]

### Design decisions worth review

1. [Decision + rationale + link to DECISIONS.md entry if applicable]
2. ...

### Testing

- Unit tests: X new, Y total (all passing)
- Integration tests: X new, Y total (all passing)
- Manual smoke test: [describe what you ran, or "pending"]

### Not done in this round (and why)

[Explicit list of scope items deferred. For each: what, why, where it goes.]

### Open questions for the user

1. [Specific question with options if you have a view]
2. ...

### Next suggested round

[One-paragraph recommendation of what should come next, or multiple options if there's a real fork.]
```

## Rules for writing a good report

### Specificity over summary

Bad:
> Added the webhook handler and some tests.

Good:
> `src/app/api/webhooks/ingest/[sourceId]/route.ts` — POST handler using `await request.text()` to preserve HMAC bytes; delegates to `ingestLead()` with `defaultIngestDeps + buildIngestObservability(sourceId).deps`; 500 handler wraps in `Sentry.captureException` with `stage: "unhandled"`.

Specific reports are grep-able, defendable, and give the reviewer an entry point.

### Honesty about what was NOT done

This is the section users are most grateful for. "Not done in this round" prevents the "you said you did X but I can't find it" cycle. Include anything that a reasonable reader might expect to be in this round but isn't. Be explicit about deferrals.

### Decisions are one paragraph, not essays

Each design decision gets one short paragraph:

- What the decision was.
- Why this option over the alternatives.
- Link to DECISIONS.md if it's non-obvious.

If you find yourself writing more than a paragraph, it's probably a standalone DECISIONS.md entry — do that, and reference it from the report.

### Test counts should be real counts

Don't approximate. If your test runner says "23 tests, 23 passing," report that. If some are skipped, say so. If integration tests aren't passing, say so loudly (it's an open issue, not something to gloss over).

### Open questions should be actionable

Bad:
> Should we think about rate limiting?

Good:
> Rate limiting strategy — plan §8 left this stubbed behind a feature flag. Options:
> (a) keep stubbed, revisit in hardening phase;
> (b) implement now with Upstash (~1h);
> (c) implement with an in-memory limiter (not production-safe but catches runaway tests locally).
> Recommend (a). OK to keep the stub?

Actionable questions can be answered with a short reply.

### The "next suggested round" recommendation

You've been deep in the code; you have the best sense of what should come next and why. Make a recommendation, even if it's "fix these follow-ups then proceed to X." Don't leave the user guessing.

## What NOT to include

- Long stretches of code. The user can read the files. Reports are about *why* and *what changed at a high level*, not *line-by-line what the code looks like*.
- Apologies for taking time. Just deliver.
- Hedging. Don't say "I think" when you mean "I did." Don't say "probably works" — either it passes tests or it doesn't.
- Restating the plan. The plan is its own document; the report is about divergence and progress against it.

## Example (short but complete)

> ## Round B — Sources CRUD (API only)
>
> ### Delivered
>
> - `src/app/api/sources/route.ts` — `GET` list, `POST` create (returns secret once).
> - `src/app/api/sources/[id]/route.ts` — `GET` (masked), `PATCH`, `DELETE`.
> - `src/app/api/sources/[id]/rotate-secret/route.ts` — `POST`.
> - `src/app/api/sources/[id]/test-webhook/route.ts` — `POST` dry-run.
> - `src/lib/sources/service.ts` — CRUD via `dbService`, explicit `organization_id` filter.
> - `src/lib/sources/validation.ts` — schemas for mapping, custom fields, country alpha-2.
> - Unit tests: 25 new (9 suites, 117 total passing). Integration tests: 23 new (2 suites, 34 total passing).
>
> ### Design decisions worth review
>
> 1. Secret masking uses `null` sentinel in `GET` responses, not `"****"`. Non-ambiguous (see DECISIONS.md #28).
> 2. `rotate-secret` invalidates prior secret immediately; no grace period in Phase 1 (see DECISIONS.md #29).
> 3. Webhook URL built from `[base URL env var]`, not `Host` header. Preview deploys would otherwise burn ephemeral URLs into source records (see DECISIONS.md #30).
> 4. `requireRoleApi()` for JSON API routes vs `requireRole()` for page redirects. Two guards, two purposes (see DECISIONS.md #31).
>
> ### Testing
>
> - Unit: 117 total, all passing (789ms).
> - Integration: 34 total, all passing (12.96s).
> - Manual smoke: tested full create → POST signed payload → dedup detect → rotate → old secret 401 flow against `[local DB service]`. No issues.
>
> ### Not done in this round
>
> - UI (`/[orgSlug]/sources`, `/[orgSlug]/leads`) — Round C.
> - End-of-phase housekeeping (update `PROJECT_STRUCTURE.md`, smoke test doc) — Round D.
>
> ### Open questions
>
> Does `test-webhook` import `defaultIngestDeps`, or is it a standalone dry-run reusing only `applyMapping` + `normalizePhone`? Answer: standalone, in `src/lib/sources/test-webhook.ts` — 1 SELECT, zero writes. Integration test confirms zero side effects.
>
> ### Next suggested round
>
> Round C — UI (Sources pages + Leads list). Shares components, makes sense to do together.

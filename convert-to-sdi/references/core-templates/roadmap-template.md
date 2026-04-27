# ROADMAP Template

Sequences features into phases. Each phase is a **shippable increment** — at the end, the product does less but still works end-to-end.

Target length: 200–300 lines.

## Structure

```markdown
# [Product] — Roadmap

> Phased delivery plan for the MVP. Goal: ship a thin slice that proves value with real [customer/users] end-to-end, then broaden.

## Principle

Each phase is shippable. At the end of each phase, the platform is usable — just does less. This lets us validate assumptions early, before investing in polish features.

---

## Phase 0 — Foundation (Week X)

**Goal**: [Repo skeleton, auth, multi-tenancy, CI. No business features yet.]

Deliverables:
- [bullets]

**Done when**: [One sentence describing the demonstrable state.]

---

## Phase 1 — [Feature name] (Week Y)

**Goal**: [One sentence.]

Deliverables:
- [bullets]

**Done when**: [Demonstrable state.]

---

## Phase 2 — ...

...

---

## Phase N — Hardening (last phase)

**Goal**: Ship to [customer] with confidence.

Deliverables:
- Rate limiting on [critical endpoints]
- Retry/backoff on [external service] failures with alerting
- Dead-letter handling for [job system] failures
- Audit pass on RLS policies
- Audit pass on API authorization
- Load test: [target volume]
- Sentry release integration
- Runbook for super admin operations
- Data backup strategy documented
- Smoke-test suite covering the N critical flows

**Done when**: We're comfortable handing real production traffic to the platform.

---

## Post-MVP Backlog (Not Scheduled)

Captured for awareness; prioritized after MVP in production.

- [items — each one a potential future phase]

---

## Fast-Track Option (optional)

If you need to show *something* to [customer] within [short window] before completing all phases:

- [Subset of phases]
- [What's skipped]
- [What's demonstrated]

This validates the **core hypothesis** before committing to the full build.
```

## Writing tips

- **Phases are weeks, not months.** If a phase takes >2 weeks, split it. Long phases have no natural checkpoints.
- **"Done when" is demonstrable.** Someone should be able to do the demo and see whether the criterion holds.
- **Phase order has a logic.** Earlier phases unblock later ones — explicitly. If phase 3 could be phase 1, question why it isn't.
- **Backlog at the end captures deferred work.** Don't lose ideas; don't schedule them prematurely either.
- **Fast-track option is optional but valuable.** It signals flexibility — "we can prove value in 2 weeks if needed" is a good story.

## Common failure modes

- **Phases that don't ship.** If Phase 2 requires Phase 3 to be usable, it's not actually a phase — it's half a phase. Split boundary differently.
- **Too many phases.** If there are 10 phases, the plan is overcooked. 5–7 is typical for an MVP.
- **"And then a miracle happens."** A late phase with huge scope ("Phase 7: build the analytics platform") isn't a phase, it's a project.
- **No hardening phase.** Production-ready requires explicit hardening work. If it's not a phase, it'll be skipped.

## Relation to IMPLEMENTATION_PLAN

The ROADMAP lists phases with high-level goals and deliverables. The IMPLEMENTATION_PLAN_PHASE_N drills into *how* for a single phase. At MVP planning time, only Phase 1's detailed plan is generated. Plans for later phases come later, just before each starts.

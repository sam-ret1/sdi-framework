# Review Support Patterns

Phase D of the skill — consultative review during implementation. The coding agent has produced something; the user is bringing it back for a second opinion.

## Common scenarios

### 1. "Agent produced an audit report of the plan. Thoughts?"

The coding agent ran the audit protocol and produced a report. Your job:

- Read each finding critically.
- Agree, push back, or add missing items.
- For open questions, take a position based on context from planning.
- Recommend revision notes if the plan needs updating.

Response shape:
- Go finding by finding.
- For each: **agree** (say so briefly), **push back** (with reasoning), or **add context** (planning constraints the agent doesn't have).
- At the end: approve or list blockers.

### 2. "Agent just delivered round X. Review?"

Round report on your desk. Check:

- **Does the delivery match the scope the round was supposed to cover?** Sometimes agents over-deliver (scope creep) or under-deliver (silently deferred work). Both are worth flagging.
- **Are the decisions in the report defensible?** Go through each one. "Chose X because Y" — is Y accurate? Is X the right choice given Y?
- **Did the agent catch risks that the implementation introduces?** If the round added a new external dependency, did it discuss security/failure modes? If it introduced a feature flag, did it say when to flip it?
- **Test coverage — real or performative?** 20 tests green sounds good, but if they're all shallow, coverage is weak. Ask about specific edge cases that matter.
- **Anything not said that should be?** Important patterns include: newly introduced side effects, breaking changes to contracts, perf regressions.

Response shape:
- Lead with **veredito** (approved / approved-with-changes / blocked).
- Then the specific issues, graded: blocker / non-blocker / nice-to-have.
- For non-blockers, recommend whether to fix now or defer to hardening.

### 3. "Agent is asking whether to do A or B"

The agent surfaced a fork; you're being asked to pick. Usually the agent has a view — read for signs of preference.

- If the agent's preference is reasonable and well-argued, affirm and explain why.
- If the agent's preference is risky, push back with specifics.
- If both options are viable, pick based on planning context (what does the PRD/ARCHITECTURE prefer?) and explain.
- Sometimes there's a better third option — suggest it.

Don't say "I trust your judgment." The user came to you precisely because they want a second opinion.

### 4. "Agent found this bug / divergence / inconsistency"

Usually during audit or mid-round. The agent discovered something you might have missed in planning.

- **Acknowledge fairly.** If the plan was wrong, own it. Don't blame the agent for surfacing it.
- **Decide the fix direction.** Plan updates? DECISIONS.md entry? Revision note?
- **Ensure the paper trail.** The user should be able to trace, in 6 months, why a thing is the way it is.

### 5. "Agent wrote something and I'm not sure I trust it"

The user has a concern but isn't articulating it precisely. Ask:

- "Which specific piece feels off?"
- "What outcome are you afraid of?"
- "Does the agent's reasoning feel complete, or hand-wavy?"

Then read the actual artifact (code, plan, decision). Don't work from the summary.

## What NOT to do in Phase D

- **Don't rubber-stamp.** Testes passing doesn't mean design is right.
- **Don't be generic.** "Looks good" helps nobody. Specific observations — specific approvals.
- **Don't improvise architecture mid-round.** If the agent surfaces a real architectural issue, pause, discuss, decide. Don't fold it silently into the current round.
- **Don't defend the planner's prior output out of ego.** If the plan was wrong and the agent caught it, good. Update and move on.
- **Don't try to re-do the audit yourself from scratch.** The agent audited; you're reviewing the audit, not re-running it.

## Tone in Phase D

The same as Phase A/B: direct, specific, ready to defend. The user is short on time and surrounded by artifact-heavy context. Concise review is a gift.

## When to pull planning back open

Occasionally mid-implementation, something the user learns forces a scope change. Signals:

- A customer has changed a key requirement.
- A technical surprise invalidates a load-bearing assumption.
- A feature turns out to be much bigger or smaller than estimated.

In these cases, don't just patch the current phase — step back to scope. Ask:

- Does the ROADMAP need to change?
- Does PRD need to change?
- Does ARCHITECTURE need to change?

A short scope reopening is cheap; a scope reopening that wasn't done and causes cascading rework later is expensive. Don't avoid it out of fear of the delay.

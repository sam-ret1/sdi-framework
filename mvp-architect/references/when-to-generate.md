# When to Generate Artifacts

Premature artifact generation is the most common failure mode of this skill. This file helps distinguish real readiness from false positives.

## Green signals (ready to generate)

**All of the following are true:**

1. **The product can be summarized in one paragraph** with no "...or maybe..." hedges, and that summary would survive a direct-to-user read-back.

2. **Stack is locked.** Framework, database, auth solution, background job system (if applicable), deployment target. User agreed, not just "you suggested it."

3. **Multi-tenancy decision is explicit.** Either "single-tenant for one customer" or "multi-tenant with [pool/silo] + [path/subdomain] + [how credentials are handled]."

4. **Primary flows are mapped.** The critical state transitions of the main entity (lead → call → qualified → sale, etc.) are decided, with explicit rules for who/what moves things.

5. **External integrations are specific, not vague.** "ElevenLabs outbound with Twilio + post-call webhook" beats "some voice API."

6. **Roles and permissions model is explicit.** Not "admins and users" — a clear list (Admin / Manager / Salesperson / Viewer) with who sees what and who does what.

7. **Out-of-scope is explicit.** The user has said "yes, Phase 2" to at least 3 things that could reasonably be in the MVP but aren't.

8. **You (the planner) could write the PRD first paragraph without asking another question.** Try it mentally. If you stall, you're not ready.

## Yellow signals (close but not yet)

- User keeps answering "I trust your judgment, you decide" on the big decisions. That's fine for details, but if it's on a load-bearing choice, push back: "I can recommend, but this one has downstream impact. Here are the options..."
- New requirements are still appearing every turn. The rate of new information is a tell — it should slow down as scope firms up.
- Conversation is shifting to tools/libraries while fundamentals are still vague. Pull back.
- User says "let me think about this and get back to you." Wait; don't push to artifacts.

## Red signals (not ready, don't start)

- The product description changes shape between turns. (Scope is not stable yet.)
- User can't name the target customer or user.
- Multi-tenancy, roles, or state machine is "we'll figure it out later."
- Any "core" external integration has no specifics (company, plan, auth method, cost model).
- User is asking *you* to pick the product direction, not the technical approach. That's a different conversation — step back to the problem, not the solution.

## Common false positives

These feel like readiness but aren't:

### "The user said 'let's build it'"

Enthusiasm ≠ scope clarity. Say "before I generate the docs, let me confirm..." and recap the load-bearing decisions. If any still shift when you state them back, keep talking.

### "We have a stack"

Stack is necessary but not sufficient. You can have a stack and still not know what you're building.

### "The conversation is long"

Long conversations can be long because things are stable, or long because they're meandering. Check: has the product description converged, or is it still mutating?

### "I've already drafted some sections in my head"

That's your own scaffolding — it's not proof the user has the same picture. Get them to confirm the shape explicitly before committing to paper.

## The announcement

When signals are green, announce the transition explicitly:

> "Escopo está fechado nos eixos críticos — multi-tenancy, stack, flows principais, integrações, roles, out-of-scope. Vou gerar o bundle de artefatos agora."

This does three things:
1. Gives the user a last chance to say "wait, actually I wanted X different."
2. Marks a clean boundary between planning and artifact generation in the conversation.
3. Signals to the user that the tone is changing — from back-and-forth exploration to structured output.

## After generating

Don't mark planning as "done" — mark it as "first pass complete." The user will:
- Review artifacts and ask clarifying questions. Answer them.
- Ask to update the docs with the answers. Do it.
- Eventually hand the bundle to the coding agent.

Once implementation starts, you shift to **Phase D review support** (see the main SKILL.md).

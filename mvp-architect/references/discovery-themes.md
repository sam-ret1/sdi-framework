# Discovery Themes (Universal)

A product idea has many possible axes to discuss. The themes below apply across all project types. For type-specific themes (multi-tenancy for SaaS, SEO for landing pages, data sources for dashboards, etc.), also load `project-types/{type}/discovery.md` after Phase 0.

If the AI/LLM modifier was activated in Phase 0, also load `modifiers/ai.md` for additional themes about model choice, eval, prompt management, and cost.

Ask questions **grouped by theme**, not as a flat list. Within each theme, ask 2–3 questions at a time. Themes can be covered in any order, but the ones marked ⚡ tend to block everything else — get those answers first.

## ⚡ 1. Data sources and integrations

Where does data enter, and where does it leave? This shapes API contracts, webhook handlers, auth flows, ingestion pipelines.

Typical questions:
- What systems feed data in (forms, partners, integrations, scrapers, user input)?
- One schema or many (e.g. different lead forms with different shapes, multiple data sources with different formats)?
- Synchronous call-and-wait, or async via webhooks/queues?
- What systems need to receive data from this product (CRMs, notification services, downstream APIs)?
- Does the user want to configure new integrations via UI, or is it always a developer handoff?

## ⚡ 2. Core flows and stateful entities

What are the main things the product tracks, and how do they move through states? This surfaces the data model and the funnel/workflow structure.

Typical questions:
- What's the primary entity (lead, order, ticket, project, document, conversation, etc.)?
- Does it have states/stages it progresses through? What are they?
- Who moves it between states — users manually, automation, or both?
- Is the progression strict (state machine) or freeform?
- Multiple parallel flows, or just one?

## 3. Users and roles

Who uses this, and with what permissions? Drives auth design, access control, UI gating.

Typical questions:
- Who are the human users? (roles/titles)
- Are there role tiers? (admin, manager, worker, viewer)
- Read vs write permissions — who sees what, who edits what?
- Any cross-context roles (super admin, support agents)?
- Is there a non-human user? (API clients, automation accounts, agents)

## 4. External service integrations (specific)

If the product depends heavily on one external service (OpenAI, Stripe, Twilio, ElevenLabs, SendGrid, scraping APIs, etc.), get into the specifics. These often dictate architecture.

Typical questions:
- Which specific API/SDK is being used?
- Synchronous calls vs async (webhooks, polling)?
- What does the request/response contract look like?
- Retry policy, rate limits, error handling?
- Cost model — per call, per minute, subscription?
- Sandbox/test mode available?
- Does the external service hold the data, or do we mirror it?

## 5. Workflows and orchestration

Beyond simple request-response, does the product have async work — retries, scheduled jobs, multi-step pipelines?

Typical questions:
- Does anything need to happen "later" (5 minutes, 1 hour, next business day)?
- Are there sequences that span multiple external calls with state in between?
- What's the retry policy for failures?
- Background jobs, cron, workflow engine (Inngest, Temporal, Trigger.dev), or just inline async?
- What happens if the system restarts mid-flow?

## 6. Non-functional requirements

The boring but critical ones. Often underspecified until the user hits a wall.

Typical questions:
- Scale: expected peak events per minute / month?
- Compliance: GDPR, LGPD, HIPAA, SOC2? Any data residency rules?
- Audit trail: need to log who did what, when?
- Data retention: how long to keep records/logs/recordings?
- Uptime expectations?

## 7. Tech stack

Leave this until the other themes are mostly clear — stack choice depends on what we're building, not the other way around. Specific stack questions live in `project-types/{type}/discovery.md` because choices vary heavily by type (web framework vs mobile framework vs data pipeline runtime vs agent SDK).

Universal stack questions:
- Language preference if more than one option (TypeScript default for web, Python for ML/data, etc.)?
- Any existing code the new project must coexist with?
- Deployment target preference (managed PaaS, self-hosted, serverless, edge)?
- Team familiarity — are there technologies the user is *not* willing to use?

## 8. UX considerations (when applicable)

Skip this theme entirely for projects without UI (API services, data pipelines, automation workflows without a console). For projects with UI, pull questions from here as the conversation matures.

Typical questions:
- Primary interaction pattern — table-heavy, form-heavy, kanban, dashboard, conversational, immersive?
- Real-time updates needed, or is refresh-on-load fine?
- Mobile usage expected, or desktop-only? (or mobile-only for mobile apps)
- Design reference apps the user likes?
- Dark mode required?
- Internationalization — single language at MVP, multi-language later?

## 9. Feature boundaries (what's NOT in the MVP)

As the scope takes shape, be explicit about what's being deferred. Users often think "this is in the MVP" for things you assumed were Phase 2.

Typical questions:
- Is billing/subscription/monetization in MVP, or manually managed for first customer?
- Is onboarding self-service, or you onboard each customer manually?
- Multi-language UI at launch, or single language only?
- Advanced analytics, or basic metrics only?
- SSO, or simple auth for now?
- Polished marketing surface, or admin-grade UI for MVP?

## How to use this reference

1. **Phase 0 already fixed the project type.** This file gives you universal themes; type-specific themes come from `project-types/{type}/discovery.md` (load that file alongside this one).

2. **First reply to the user:** pick 3–5 themes most critical for the described product. Universal themes 1, 2, and one of {3, 4, 5} usually anchor the start. Type-specific themes load right after.

3. **Group questions.** Don't ask every question in a theme — pick the 2–3 most load-bearing.

4. **As the conversation progresses:** introduce new themes based on what's emerging. If they describe a voice-call feature, pull in theme 4 (specific external service) and theme 5 (workflows).

5. **Don't ask themes that don't apply.** Use judgment. A simple landing page skips theme 5 entirely.

6. **Track which themes are closed vs open.** When all critical themes are closed and the user is converging, check `when-to-generate.md`.

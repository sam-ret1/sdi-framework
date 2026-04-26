# Discovery — Web SaaS multi-tenant

Type-specific themes for web-based multi-tenant SaaS products. Load alongside `references/discovery-themes.md`.

## ⚡ 1. Multi-tenancy model

Shapes almost everything downstream (data model, auth, isolation, deployment, billing). Get this clear early. Ask even if the user describes the product as "for one customer" — SaaS that might expand later has very different architecture than SaaS locked to one tenant.

Typical questions:
- Is this truly single-tenant or could it expand to other customers?
- If multi-tenant, pool model (one DB, tenant column, row-level isolation) or silo (one DB per tenant)?
- Path-based (`app.com/org-slug/...`) or subdomain-based (`org.app.com/...`)?
- Credentials to third-party services: platform-owned and shared across tenants, or tenant-owned?
- Tenant onboarding: self-service signup or manual provisioning?

## 2. Auth and identity

Drives sign-up, sign-in, role gating, session management.

Typical questions:
- Email + password, magic link, OAuth (Google/Microsoft), SSO (SAML/OIDC) — which combinations?
- Identity provider — managed (Auth0, Clerk, Supabase Auth, WorkOS) or self-built?
- Tenant invitations: how do new users join an existing tenant?
- Password reset, email verification — must-have at MVP or later?
- Session model — short-lived JWT, refresh tokens, server sessions, sticky cookies?

## 3. Roles and permissions within a tenant

Drives access control, UI gating, audit trails.

Typical questions:
- What role tiers (admin, manager, worker, viewer)? Are these fixed or configurable per tenant?
- Read vs write permissions — who sees what, who edits what?
- Cross-tenant roles (super admin, support agents, impersonation)?
- Per-resource ownership rules?
- Audit trail requirements — log who did what, when?

## 4. Data isolation strategy

Drives the actual enforcement of multi-tenancy. Critical for security and compliance.

Typical questions:
- Row-level isolation in the DB (e.g. policies, tenant_id checks) — primary defense?
- Application-layer enforcement only (middleware checks) — acceptable risk?
- Service-role bypass needed anywhere (admin tools, jobs)? What compensating controls?
- Read-replica access? Per-tenant analytics vs cross-tenant reporting?

## 5. Billing and monetization

Often deferred to Phase 2+, but the model affects data design.

Typical questions:
- Free tier, trial, freemium, paid-only?
- Per-seat, per-tenant, per-usage, hybrid?
- Provider preference (Stripe, Lemon Squeezy, Paddle, custom)?
- Invoicing/tax handling (Stripe Tax, Lago, manual)?
- Cancellation/downgrade flow — graceful or strict?

## 6. Onboarding and lifecycle

Often where SaaS ships dies — from signup to first value to retention.

Typical questions:
- What's the "first value moment" — what does the user need to see/do for the product to feel useful?
- Self-service onboarding or guided (sales call → manual setup)?
- Empty-state handling — sample data, demo tenant, walkthroughs?
- Email/notification triggers across the lifecycle (welcome, drip, churn warning)?

## 7. Super admin and operations

Often forgotten until a customer issue forces it.

Typical questions:
- Need to view/edit data across tenants for support?
- Impersonation for debugging customer issues?
- Bulk ops (re-process leads for a tenant, delete tenant data on offboarding)?
- Operational dashboards (signups, MRR, churn, tenant health)?

## How to use

1. Theme 1 (multi-tenancy model) is the highest leverage — get it answered before Theme 2–7.
2. Themes 2–4 are tightly coupled. If a managed identity provider handles RLS for you, Theme 4 becomes shallower.
3. Theme 5 (billing) can usually be deferred to Phase 2 — but ask "is billing in MVP?" so you know whether to scaffold or skip.
4. Themes 6 and 7 are easily forgotten. Ask at least one question from each before declaring scope closed.

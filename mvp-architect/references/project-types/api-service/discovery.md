# Discovery — Web API / backend service

Type-specific themes for headless services: REST/GraphQL APIs, webhook receivers, microservices, internal RPCs. Load alongside `references/discovery-themes.md`.

## ⚡ 1. Consumer model

Who calls this service, and how?

Typical questions:
- Internal-only (called by your own frontend/services), partner-facing (third parties), public (developer ecosystem), or mix?
- Synchronous request/response or async (queue, webhook callback)?
- Single consumer or many?
- Versioning required at MVP (e.g. `/v1/`) or single-version with deprecation later?
- SDK distribution — auto-gen from OpenAPI, hand-written, or just docs?

## ⚡ 2. Auth and authorization

Drives gateway pattern, key management, rate limiting tiers.

Typical questions:
- Auth model — API keys, JWTs, OAuth 2.0 client credentials, mTLS, signed requests?
- Per-key scopes / per-user permissions?
- Key rotation, revocation, last-used tracking?
- Webhook auth (incoming) — HMAC signatures, IP allow-lists, both?
- Multi-tenant: how is the tenant resolved per request?

## ⚡ 3. Endpoint surface

What's the API actually offering?

Typical questions:
- REST, GraphQL, RPC (gRPC/tRPC/JSON-RPC), or hybrid?
- Resource shape — coarse-grained domain endpoints or many fine-grained?
- Read vs write balance — mostly reads, mostly writes, balanced?
- Bulk operations — yes/no, with what idempotency story?
- Pagination strategy — cursor, page+limit, since-token?

## 4. Persistence

Most APIs need to read/write something.

Typical questions:
- Database — relational, document, key-value, search index, multiple?
- Schema management — migrations, CDC, dbt?
- Read scaling — replicas, caching layer?
- Write semantics — strong consistency, eventual, transactional across resources?
- Data retention and deletion (especially for compliance)?

## 5. Webhooks (incoming)

If this service receives webhooks from third parties.

Typical questions:
- Which providers send webhooks (Stripe, Twilio, GitHub, custom partners)?
- Signature verification scheme per provider?
- Replay protection — request id de-dup, timestamp tolerance?
- Processing model — sync within webhook handler, or queue + ack-quickly?
- Retries from upstream — what's the upstream's policy, and how does our handler stay idempotent?

## 6. Webhooks (outgoing)

If this service emits webhooks to consumers.

Typical questions:
- Consumer registration — UI, API, both?
- Delivery guarantees — at-least-once with retry, max attempts?
- Backoff schedule — exponential, ceiling?
- Signature scheme for consumers to verify?
- Failure handling — DLQ, dashboard, email alerts?

## 7. Rate limiting and quotas

Prevents abuse and aligns with cost/SLA tiers.

Typical questions:
- Per-key, per-IP, per-tenant — which dimensions?
- Tier-based (free/pro/enterprise) or single-tier at MVP?
- Limit enforcement — token bucket, leaky bucket, fixed window?
- Response when limited — 429 with Retry-After, queued, soft-fail?
- Where state lives — Redis, in-memory (single-node only), DB?

## 8. Observability and SLA

What "healthy" means for this service.

Typical questions:
- p50/p95/p99 latency targets per endpoint?
- Error rate budget?
- Uptime target — 99.9, 99.99?
- Log destination — what's logged, with what PII scrubbing?
- Metrics — request rate, error rate, latency histogram by endpoint?
- Tracing — OpenTelemetry, vendor (Datadog/Honeycomb), none at MVP?

## How to use

1. Themes 1, 2, 3 are foundational. Without these, endpoint design is guesswork.
2. Theme 4 (persistence) couples tightly to ARCHITECTURE — answer it before sketching the data model.
3. Themes 5 and 6 (webhooks) often eat half the engineering effort if present. Don't underestimate.
4. Theme 7 (rate limiting) is frequently deferred. Ask whether MVP needs real enforcement or a stub.

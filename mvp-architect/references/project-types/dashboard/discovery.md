# Discovery — Dashboard / admin / internal tool

Type-specific themes for products whose primary surface is data visualization, internal admin, or operational tooling. Load alongside `references/discovery-themes.md`.

## ⚡ 1. Audience and access

Who uses this, and what does the access model look like?

Typical questions:
- Internal team only, customers, both, or one of those plus a small group of partners?
- Single org or multi-org (multi-tenant)?
- Roles — viewer-only, editor, admin? Per-resource permissions or org-wide?
- SSO required at MVP, or simple auth acceptable?
- Hide-by-default vs show-by-default — does role drive visible features?

## ⚡ 2. Data sources

Where does the data come from, and how fresh does it need to be?

Typical questions:
- Single source (one DB) or multi-source (DB + warehouse + 3rd-party APIs)?
- Direct DB queries, materialized views, dedicated read replica, dbt models?
- Real-time (live counter that updates as events arrive), near-real-time (1-min latency), or batch (hourly/daily refresh)?
- Aggregation tier — is the dashboard reading raw rows or pre-aggregated summaries?
- Read scale — how many concurrent users querying, how heavy are the queries?

## ⚡ 3. Refresh and caching

Determines architecture and UX.

Typical questions:
- Default refresh — on-load, polling every N seconds, push via websocket?
- Cache layer — server-side query cache (Redis/Memcache), CDN edge cache, browser cache?
- Stale-while-revalidate semantics or hard expiry?
- Manual refresh button — yes/no, with what feedback?

## 4. Visualizations

The actual content of a dashboard.

Typical questions:
- Default chart kit — line, bar, area, pie, table-with-sparklines, KPI tile, funnel?
- Library preference — Recharts, Tremor, Visx, ECharts, Plotly, custom?
- Interactive features — hover details, click-to-drill, brush selection, cross-filter?
- Comparison mode — date-range comparison (this week vs last week), cohort comparison?
- Export — PNG/PDF/CSV, who can export, with what logging?

## 5. Filters and segmentation

How users slice the data.

Typical questions:
- Filter dimensions — date range, user/org, status, custom field?
- Saved views / bookmarks — per-user persisted filters?
- URL-as-state — shareable links that encode filters?
- Default filter — "last 7 days" or similar, with override?

## 6. Operational actions (if admin/internal tool)

Dashboards that mutate state need clear rules.

Typical questions:
- Read-only dashboard, or edit/admin actions inline?
- High-blast actions (delete, refund, suspend) — confirmation flow, audit log, two-person rule?
- Bulk operations — yes/no, with what guardrails?
- Action history — visible to user, queryable by admin?

## 7. Performance and pagination

Dashboards die of slow queries.

Typical questions:
- Largest table size — 10k rows, 1M, 100M?
- Pagination strategy — cursor, offset, virtual scroll?
- Aggregation budget — max query time before timeout/fallback?
- Slow-query handling — show skeleton, partial results, error?

## 8. Exports and sharing

Often added late and badly.

Typical questions:
- CSV export — straightforward; what's the row limit?
- Scheduled email reports — daily summary digest?
- Shareable read-only links to a filtered view?
- Embed via iframe in another product?

## How to use

1. Themes 1, 2, 3 are foundational. Get those answered before sketching the UI.
2. Theme 4 is where the design starts; expect long discussion.
3. Theme 6 (operational actions) is often deferred. Ask whether MVP is read-only or includes mutations.
4. Theme 7 (performance) is the silent killer. Address it in scope, not after launch.

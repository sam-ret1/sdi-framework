# Discovery — Landing page / marketing site

Type-specific themes for marketing sites and lead-gen landing pages. Load alongside `references/discovery-themes.md`.

## ⚡ 1. Goal and conversion target

Landing pages exist to drive a single action. Without clarity on the action, the design is rudderless.

Typical questions:
- Primary CTA — sign up, book a demo, download lead magnet, install app, buy?
- Secondary CTAs — newsletter, docs, pricing? How prominent vs primary?
- Conversion target — what conversion rate are you optimizing toward, from what traffic volume?
- Lead capture fields — email only, or full qualification (name, company, role, intent)?
- Where does the lead/conversion go — CRM, email list, Slack, Sheet?

## ⚡ 2. Content sources and updates

Who edits content, how often, and where does it live?

Typical questions:
- Static (committed in repo) or CMS-driven (Sanity, Notion, Contentful, headless WP)?
- Who updates copy — devs only, marketing team self-service?
- Update cadence — major redesigns quarterly, copy tweaks weekly?
- Multi-language at launch or single language?
- Blog/content section in scope, or just core pages?

## ⚡ 3. SEO and discoverability

Often the actual product KPI for marketing sites.

Typical questions:
- Target keywords — known and prioritized, or research still pending?
- Sitemap and robots strategy — single sitemap, segmented, dynamic?
- Open Graph / Twitter cards — per-page custom or single template?
- Schema.org structured data — Product, Organization, Article, FAQ?
- AMP, mobile-optimized, Core Web Vitals targets?

## 4. Performance and infra

Performance directly affects conversion (LCP < 2.5s is roughly minimum).

Typical questions:
- Hosting — Vercel, Cloudflare Pages, Netlify, custom CDN?
- Build strategy — fully static, ISR, on-demand revalidation?
- Image strategy — `next/image`, custom CDN, responsive sources?
- Analytics — what tool, what events? GA4, PostHog, Plausible, Fathom?
- Cookie/consent banner — required by jurisdiction, what tool?

## 5. Forms and integrations

Forms are usually where backend complexity creeps in.

Typical questions:
- Where do form submissions go (HubSpot, Mailchimp, Salesforce, Sheets, custom API)?
- Validation — client-side only, server-side, both?
- Spam protection — honeypot, hCaptcha, Cloudflare Turnstile, rate limiting?
- Confirmation UX — inline message, redirect to thank-you, email confirmation?
- Webhook chains for downstream automation (Zapier, n8n, custom)?

## 6. A/B testing and personalization

Often deferred but sometimes Phase 1 if marketing team is sophisticated.

Typical questions:
- A/B test in MVP or later? If yes, which tool (Vercel A/B, GrowthBook, custom)?
- Personalization — geo, traffic source, returning visitor — needed at MVP?
- Feature flags for content variants?

## 7. Compliance and legal pages

Easily forgotten until launch.

Typical questions:
- Privacy policy, ToS, cookie policy — drafted, who owns?
- GDPR/LGPD compliance — cookie consent, data subject requests, regional handling?
- Accessibility floor — WCAG 2.2 AA?

## How to use

1. Themes 1, 2, 3 are the highest leverage. Without a sharp answer to Theme 1, the rest is decoration.
2. Theme 4 (perf) is rarely negotiable for marketing sites. Establish targets early.
3. Theme 5 (forms) is where MVP often fails — keep the integration list short and well-tested.
4. Themes 6 and 7 are usually deferrable but ask whether they're in MVP or later.

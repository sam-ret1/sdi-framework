# DESIGN_SYSTEM Template — Landing page / marketing site

Marketing sites have specific design pressures: brand expression matters more than density, hero treatments are the single largest design decision, and conversion-driven UI components are a fixed kit (hero, social proof, CTA, pricing, FAQ).

Target length: 200–350 lines.

## Structure

```markdown
# [Product] — Design System

> Visual language for the marketing site.

## 1. Brand intent

[One or two paragraphs. What does the site need to feel like in 3 seconds? Confident vs playful, technical vs friendly, premium vs accessible. Reference brands or sites whose tone you admire (Linear, Vercel, Stripe, Apple) — for tone, not literal copying. What to avoid (e.g. "do not feel enterprise-stodgy", "do not feel template-y").]

## 2. Typography

- **Display font:** [name + weight] — used for hero headlines, section titles
- **Body font:** [name + weight] — used for paragraphs, lists
- **Monospace (if applicable):** [name] — used for code/data showcases
- **Type scale:** explicit ladder (e.g. 12, 14, 16, 18, 24, 32, 48, 64, 80) with usage notes
- **Line height rules:** tight for display, comfortable for body

## 3. Color system

- **Brand:** primary, accent
- **Neutrals:** full ramp from background to high-contrast foreground
- **State:** success, warning, danger (limited use on marketing sites — typically only in form feedback)

### Token definitions

\`\`\`css
:root {
  --bg: ...;
  --bg-muted: ...;
  --fg: ...;
  --fg-muted: ...;
  --primary: ...;
  --primary-fg: ...;
  --accent: ...;
  --border: ...;
}

.dark {
  /* dark mode values, if applicable */
}
\`\`\`

### Semantic usage

| Token | Where used |
|---|---|
| `--primary` | Primary CTAs, focused link states |
| `--accent` | Secondary CTAs, badges, highlights |
| `--bg-muted` | Section backgrounds for visual rhythm |
| ... | ... |

## 4. Spacing and rhythm

- **Section padding:** consistent vertical rhythm across the page (e.g. 96px top/bottom on desktop, 64px on tablet, 48px on mobile)
- **Container width:** max content width (e.g. 1200px) with horizontal padding rules
- **Grid:** 12-col desktop, 6-col tablet, 4-col mobile

## 5. Hero treatments

The single most consequential visual decision on a marketing site.

- **Default hero pattern:** [centered headline + sub + CTA + product visual] / [split (text left, visual right)] / [full-bleed video] / [animated illustration]
- **Visual asset rules:** video formats and weights, fallback for slow connections, motion-reduce honor
- **Headline rules:** max length, allowed line breaks, weight

## 6. Sections kit

Marketing sites are largely the same kit of sections. State the canonical patterns:

- **Section header** (eyebrow + title + lede)
- **Feature grid** (icon + title + description, 2/3/4-col responsive)
- **Pricing table** (3-tier classic, with most-popular highlight)
- **Testimonial / social proof** (logo bar, quote card, case-study link)
- **FAQ** (accordion, single-open or multi-open)
- **CTA section** (single primary action, sometimes secondary)
- **Footer** (sitemap, legal, social, optional newsletter)

Each pattern has fixed spacing, type scale, and behavior — variations are explicit, not improvised.

## 7. Motion

- **Defaults:** entrance fades on scroll (subtle, ~300ms ease-out)
- **Hero motion:** allowed if it serves the headline; never decorative
- **Interaction motion:** button hovers, link underlines — restrained
- **Reduced-motion:** all animations have a static fallback

## 8. Iconography and imagery

- **Icon library:** [Lucide / Heroicons / custom set] with stroke weight rule
- **Photography style:** product UI shots, brand photography, no stock-feeling templates
- **Illustration style:** [if used — flat geometric, isometric, hand-drawn]

## 9. Forms

Forms on marketing sites are conversion-critical.

- **Field layout:** label above, input full-width, error inline below
- **Validation feedback:** real-time after first blur, never on every keystroke
- **Success state:** clear, celebratory but brief
- **Loading state:** disabled CTA + spinner

## 10. Accessibility

- **WCAG 2.2 AA** as floor (most marketing sites) or AAA target if brand demands
- **Focus rings:** visible on all interactive elements; brand-colored if possible
- **Color contrast:** body text ≥ 4.5:1, large display ≥ 3:1
- **Heading hierarchy:** semantic H1–H6 enforced; no header-as-style abuse
- **Alt text:** required on all content images; decorative images use `alt=""`

## 11. Implementation notes

- **CSS strategy:** [Tailwind / vanilla-extract / CSS modules + tokens / CSS-in-JS]
- **Component primitives:** [shadcn / Radix / Headless UI / custom]
- **Where tokens live:** `src/styles/tokens.css` is the source of truth; tailwind.config / theme config imports them
- **Font loading:** framework-native font optimization; no flash of unstyled text
```

## Writing tips

- **Hero section deserves disproportionate attention.** It's where 80% of first impressions happen.
- **Sections kit is finite.** Define it; resist inventing new sections without a strong reason.
- **Motion guidance restrained.** Marketing sites suffer from too much animation, not too little.
- **Tokens align to actual code.** Don't document tokens that the code doesn't use.

## Common failure modes

- **Hero by committee.** Multiple visual ideas crammed into one. Pick one pattern, execute well.
- **Section sprawl.** Inventing one-off sections instead of reusing the kit, leading to inconsistent rhythm.
- **Stock-photo feel.** Generic imagery undermines premium positioning.
- **Performance via sacrifice.** Cutting visual quality to hit perf scores; better to optimize delivery than reduce design.

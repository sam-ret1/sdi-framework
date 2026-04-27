# DESIGN_SYSTEM Template

Feeds both prototyping tools (Claude Design, Figma) and implementation (coding agent applying Tailwind tokens, shadcn wrappers, or equivalent).

Include only when the project has a UI.

Target length: 250–400 lines.

## Structure

```markdown
# [Product] — Design System

> Visual language and token specification for the platform.

## 1. Design Intent

[One or two paragraphs. What should the UI feel like? Aesthetic reference points (Linear, Attio, Height, Retool) — for tone, not literal copying. What to avoid.]

## 2. Typography

[Font families + weights + usage. Type scale in Tailwind classes or equivalent. Line height rules.]

## 3. Color System

[Brand tokens: primary, accent/success, warning, destructive, neutrals.]

### Token Definitions

\`\`\`css
:root {
  --bg: ...;
  --fg: ...;
  --fg-muted: ...;
  --primary: ...;
  /* etc */
}

.dark {
  /* dark mode values */
}
\`\`\`

### Semantic Usage Map

[Table: token → where it's used. "`--primary` is used for primary actions, links, active tabs."]

## 4. Spacing & Layout

[Padding defaults, content widths, sidebar dimensions.]

## 5. Radii & Borders

[Border radius defaults per component class. Shadow usage — usually: restrained.]

## 6. Motion

[Transition duration defaults. When to animate, when not to. Reduced-motion respect.]

## 7. Iconography

[Library (Lucide, Heroicons, etc.), stroke weight, sizes.]

## 8. Data Visualization

[Chart library, color palette discipline for series, default chart types per purpose.]

## 9. Accessibility

[WCAG target, focus rings, color-is-not-only-indicator rule, prefers-reduced-motion, contrast ratios.]

## 10. Component Patterns

[App-specific composites: lead cards, metric cards, activity timelines, test consoles. Brief description of each with notable details.]

## 11. Implementation Notes

[Practical notes for the coding agent: which primitives to use (shadcn vs Radix wrappers vs raw), where CSS lives, fonts import strategy.]
```

## Writing tips

- **Aesthetic intent matters.** "Quiet confidence" sets a direction; "modern and clean" doesn't.
- **Tokens must match what the code will actually use.** If the repo uses `--bg` / `--fg-muted`, don't document `--background` / `--foreground-muted`. Align from day one, or update at end of phase.
- **Semantic map clarifies what each token means.** Without it, developers pick tokens by color instead of meaning.
- **Motion guidance should be restrained.** Most SaaS UI suffers from too much animation, not too little.

## Common failure modes

- **Shadcn/tailwind token names divergent from actual repo.** Be explicit — the doc names the tokens the *code actually uses*, even if they differ from framework defaults.
- **Over-designing.** MVP doesn't need 12 component variants. Define what's needed; add more as demand appears.
- **Missing accessibility.** WCAG 2.2 AA is the floor — state it and mean it.
- **Doc drifts from code.** End-of-phase housekeeping should include an audit of tokens used in `globals.css` vs tokens documented here.

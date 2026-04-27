# DESIGN_SYSTEM Template — Mobile app

Mobile design has specific pressures: touch targets, platform conventions (iOS HIG vs Material), variable screen sizes, dark mode is non-optional, system font scaling matters.

Target length: 250–400 lines.

## Structure

```markdown
# [Product] — Design System

> Visual language for the mobile app.

## 1. Platform posture

State your stance on platform conventions:

- **iOS-leaning:** UI follows iOS Human Interface Guidelines (segmented controls, SF Symbols, large titles), Android is adapted but doesn't try to look fully Material
- **Android-leaning:** Material 3, Material You where applicable, iOS gets adapted treatment
- **Cross-platform unified:** custom design language shared across both, accepting the trade-off of looking slightly off-platform on each

## 2. Typography

- **System fonts:** [SF Pro on iOS, Roboto on Android] / [single custom font everywhere]
- **Type scale:** Title 1, Title 2, Headline, Body, Callout, Subhead, Footnote, Caption — explicit pt sizes
- **Dynamic type:** support system font scaling on iOS (mandatory for accessibility); equivalent on Android
- **Line height:** comfortable for body, tight for titles

## 3. Color system

- **Brand:** primary, accent
- **Neutrals:** ramp tuned for both light and dark mode
- **Semantic:** success, warning, danger
- **Both modes from day one:** dark mode is mandatory on mobile, not optional

### Token definitions

\`\`\`
LightTheme:
  bg, bgElevated, bgMuted
  fg, fgMuted, fgSubtle
  border, borderStrong
  primary, primaryFg, accent
  success, warning, danger

DarkTheme: (parallel keys with adjusted values)
\`\`\`

### Semantic usage

| Token | Where used |
|---|---|
| `bgElevated` | Cards, modals, sheets |
| `border` | Input fields, list separators |
| `primary` | Primary CTA, focused state, current selection |
| ... | ... |

## 4. Spacing and layout

- **Grid base:** 4 or 8 (commit to one)
- **Edge insets:** standard horizontal padding (e.g. 16dp)
- **Safe areas:** every screen respects notch, home indicator, status bar
- **Touch targets:** minimum 44pt × 44pt (iOS HIG) / 48dp × 48dp (Material)

## 5. Radii and elevation

- **Border radius:** moderate — 8 for cards, 12 for sheets, 999 for pills
- **Elevation:** subtle shadows; on Android use Material elevation; on iOS shadows are softer

## 6. Components

### Navigation
- **Tab bar:** count (3-5 typical), icon + label, current state treatment
- **Stack header:** large title vs compact, back button, right-side actions
- **Bottom sheet vs modal:** when each is used

### Inputs
- **Text field:** size, focus state, error state, helper text
- **Selectors:** picker / segmented control / dropdown — what's used where
- **Toggles:** native switch
- **Forms:** vertical layout, labels above, error inline

### Lists
- **Cell types:** simple, with detail, with image, with action
- **Separators:** inset to text edge or full-width
- **Pull-to-refresh:** required on list screens

### Buttons
- **Primary:** filled, full-width on action sheets
- **Secondary:** tinted or outlined
- **Tertiary:** plain text
- **Destructive:** red foreground or filled

### Feedback
- **Toast / snackbar:** transient, auto-dismiss
- **Alert:** OS-style dialog for confirmations
- **Loading:** activity indicator (centered for full-screen, inline for buttons)
- **Empty state:** illustrated, with primary action

## 7. Motion

- **Default duration:** 250-300ms
- **Easing:** native curves (iOS spring, Material standard)
- **Page transitions:** platform-default
- **Reduce-motion:** all custom animations have static fallback

## 8. Iconography

- **Library:** [SF Symbols (iOS-leaning) / Material Icons / Phosphor / custom set]
- **Sizes:** 16, 20, 24, 32 — chosen scale
- **Color:** inherits from foreground token unless explicitly themed

## 9. Dark mode

- Tested on every screen
- System-respecting by default; per-app override allowed
- Token swap; no hard-coded colors anywhere

## 10. Accessibility

- **Screen reader:** every interactive element has accessibility label
- **Dynamic type:** UI scales gracefully up to system maximum
- **Color contrast:** body ≥ 4.5:1 in both modes
- **Touch targets:** never below platform minimum
- **Voice control / switch control:** focus order is logical

## 11. Imagery and illustration

- **Style:** photographic / illustrated (flat, isometric, hand-drawn)
- **Avatar handling:** circular, fallback initials, image loading state
- **Empty-state illustrations:** consistent style across the app

## 12. Implementation notes

- **Theme provider:** wraps the app, exposes tokens via context/hook
- **Component library:** [shadcn-rn / NativeBase / Tamagui / built from scratch / Flutter Material]
- **Asset pipeline:** 1x/2x/3x for raster on iOS, density-qualified folders on Android, vector preferred where possible
- **Where tokens live:** `src/theme/tokens.ts` — single source of truth; theme provider consumes
```

## Writing tips

- **Touch targets are non-negotiable.** Below the platform minimum is a bug, not a style choice.
- **Dark mode is mandatory.** Both modes from day one, every screen tested.
- **Platform conventions matter.** Pick a posture (iOS-leaning, Android-leaning, unified) and commit.
- **Dynamic type is accessibility.** Skipping it excludes users.

## Common failure modes

- **Web-style design ported to mobile.** Hover states, tiny touch targets, no platform feel.
- **Single-mode design.** Building light first, retrofitting dark. Always parallel.
- **Hard-coded colors.** Tokens or nothing. Hardcoded values block dark mode and theming.
- **Forgotten safe areas.** Notches, home indicators, gesture areas — every screen checked.

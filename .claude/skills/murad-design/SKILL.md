---
name: murad-design
description: Use when building, styling, or reviewing UI in any project that depends on @murad/tokens — designing pages or components, choosing colour/type/spacing, wiring the stylesheet, adding a design token, or reviewing UI for consistency and accessibility. Enforces the sharp editorial identity, the expression-mode split, and the anti-patterns that break it.
---

# Murad Design System

This project uses `@murad/tokens` — one visual identity shared across several projects.

**The identity is colour, edge, and depth — not typeface.** Type and density are per-project
(the "expression mode"). Colour, radius, shadow, focus, and motion are shared and must not be
altered locally.

Full reference: `node_modules/@murad/tokens/DESIGN.md`. Read it when configuring a project
from scratch, adding a token, or resolving a conflict this file doesn't cover.

## Before styling anything

Check which mode the project imports — grep the root stylesheet for `@murad/tokens/`:

- **`editorial`** → Playfair headings, 1.75 leading, 65ch measure, breakout utilities.
  For reading: articles, marketing, docs.
- **`product`** → Inter throughout, 1.5 leading, tabular numerals, 44px tap targets,
  no measure limit. For scanning: dashboards, tables, forms, admin.

If neither is imported, the setup is incomplete — see DESIGN.md STEP 1 and 2 before
proceeding. Do not pick a mode silently; it is a judgement about what the product is.

## Rules

**Colour — tokens only.** `bg-canvas`, `bg-surface`, `text-ink`, `text-ink-muted`,
`text-accent`, `border-border`, `shadow-editorial`.

Never write a raw hex, `rgb()`, or a Tailwind palette name (`zinc-800`, `blue-500`,
`gray-500`) in component code. If the colour you need has no token, say so and propose
adding one to `core.css` — do not inline it.

**Radius — two values, nothing between.** `rounded-none` (the default for everything) or
`--radius-diecut` / `.rounded-asymmetric` for meta-layers only: callouts, modals, feature
cards. Never on buttons, inputs, or images. A 4px or 8px radius is the fastest way to make
this system look generic.

**Accent is scarce.** Coral marks the primary action and nothing else. Four coral elements
on a page means no primary action. Use `--color-success` / `--color-error` for feedback.

**No Playfair in `product` mode.** It is absent from that font stack deliberately — below
~24px its stroke contrast collapses, and dense UI is all small type. `--font-serif` resolves
to the sans stack there so ported components degrade gracefully.

**Motion.** Animate `transform`, `opacity`, `filter` only — never `width`, `height`, `top`,
`left`. Every animation needs a `prefers-reduced-motion` exit. Use `.smooth-transition`
rather than hand-rolling a transition.

**Focus.** Never `outline: none` without an equivalent replacement. Core ships a shared
focus ring; keep it.

**Touch targets (`product` mode).** Nothing interactive below 44px. Use `.tap-target`, or
`.tap-expand` when the control must stay visually small.

## Contrast — the accent has a ceiling

The coral is mid-lightness: a good fill, a bad text colour. Measured ratios:

| Pair | Ratio | Verdict |
|---|---|---|
| `accent` as text on canvas | 3.58:1 | ❌ fails body text |
| `accent-text` as text on canvas | 4.79:1 | ✅ |
| white text on accent fill | 3.74:1 | ❌ |
| `ink` text on accent fill | 4.76:1 | ✅ |

**Two rules, not interchangeable:**

- **Text ON an accent fill → `text-ink`.** A coral button takes dark ink, not white. This
  cuts against the reflex of white-on-coloured-button, but white measures 3.74:1 and fails.
- **Accent AS text → `text-accent-text`.** Resolves to the darker coral in light mode and
  standard coral in dark, because the relationship inverts between themes. Using
  `text-accent` for a body-size link is a WCAG AA failure in light mode.

`--color-accent` stays correct for fills, borders, focus rings, underlines, and display type
at 24px or above.

Check both themes before finishing, not just the one you are developing in.

## Adding a utility

Declare it with `@utility name { … }`, never inside `@layer utilities`. In Tailwind v4
only `@utility` registers a class with the variant system — a class in `@layer utilities`
works bare but `md:`, `hover:` and `dark:` prefixes on it silently generate nothing.

Nest reduced-motion guards inside the utility body so they survive variant composition,
and keep `!important` on them — without it a later same-specificity utility (e.g.
`duration-700`) defeats the guard.

Note: `delay-100`…`delay-400` collide with Tailwind core's `delay-*` (transition-delay).
Both apply, targeting different properties. Harmless, but expect both.

## Adding a token

1. Identity (colour, radius, shadow, motion constant) → `core.css`.
   Expression (size, leading, spacing, measure, density) → the mode file.
2. Register in `@theme` pointing at a distinct `--murad-*` primitive. Never
   `--color-x: var(--color-x)` — it reads as circular and breaks silently on Tailwind upgrades.
3. Define the primitive in **both** `:root` and `.dark`.
4. Write the `@rationale` comment. Every block in this package explains why it exists.

## Stylesheet wiring

Order is load-bearing:

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
@import "@murad/tokens";
@import "@murad/tokens/product";   /* exactly one mode */
@source "../node_modules/@murad/react/dist";  /* only if component pkg installed */
```

**Diagnostic:** components with correct structure and no styling at all → a missing
`@source`. Unstyled headings → no mode imported.

## Reviewing UI

Flag, in order of severity: raw hex or palette names in components · `text-accent` on
body-size text (use `text-accent-text`) · white text on an accent fill (use `text-ink`) ·
a radius between 0 and 1.5rem · missing focus state · sub-44px targets in `product` mode ·
accent used for more than the primary action · Playfair below 24px · animation without a
reduced-motion exit · a token defined in only one theme.

# @murad/tokens — Configuration & Decision Flow

Read this before writing UI in a project that depends on `@murad/tokens`.

This package carries one visual identity across many projects. **The identity is the
colour, the edge, and the depth — not the typeface.** Two projects using this package
should be recognisably related while still being appropriate to their own context. That
is the whole design of the package, and it is why the layer split below is not optional.

---

## The layer map

| Layer | File | Tunable? |
|---|---|---|
| **Identity** | `src/core.css` | ❌ Never. Shared by every project. |
| **Interaction** | `src/engine.css` | ❌ Never. Motion + scroll behaviour. |
| **Effects** | `src/effects.css` | ➖ Optional import. Use or don't. |
| **Expression** | `src/modes/*.css` | ✅ Pick exactly one per project. |

If you find yourself wanting to change something in `core.css` for one project, you have
almost certainly found a **mode** concern that is sitting in the wrong file. Fix the
placement rather than forking the value.

---

## STEP 1 — Pick an expression mode

Answer one question: **is this interface primarily read, or primarily scanned?**

| Signal | → Mode |
|---|---|
| Long-form articles, essays, case studies | `editorial` |
| Marketing and landing pages | `editorial` |
| Documentation, changelogs | `editorial` |
| Dashboards, tables, data grids | `product` |
| CRUD forms, settings, admin panels | `product` |
| Anything with a persistent nav shell around dense content | `product` |

**Tie-breaker:** if the interface has a sidebar and a data table, it is `product`, even
if it also has a marketing page. Marketing pages can be `editorial` in a *separate* app
or route group with its own stylesheet entry.

### What each mode actually changes

|  | `editorial` | `product` |
|---|---|---|
| Headings | Playfair Display, weight 400 | Inter, weight 600 |
| Body leading | 1.75 | 1.5 |
| Letter-spacing | 0.01em | 0 |
| Body size | 18px prose | 15px |
| Measure | 65ch enforced | none — container widths |
| Numerals | proportional | tabular |
| Touch targets | — | `--spacing-tap: 44px` enforced |
| Breakout utilities | ✅ | ❌ |

**Both modes share:** the coral accent, the ink/canvas/surface semantics, `--radius-sharp: 0`
with the die-cut exception, the editorial shadow language, the focus ring, and all motion.
That shared set is what makes them the same system.

---

## STEP 2 — Install and wire

```bash
npm install github:muradmadi/murad-tokens
```

In the project's root stylesheet, **in this exact order**:

```css
@import "tailwindcss";                          /* 1. engine first */
@custom-variant dark (&:where(.dark, .dark *)); /* 2. class-based dark mode */
@import "@murad/tokens";                        /* 3. identity + engine + effects */
@import "@murad/tokens/product";                /* 4. exactly ONE mode */
```

Order is load-bearing. Tailwind must initialise before `@theme` blocks are declared, and
the mode must come after core so its `@theme` additions merge rather than get overwritten.

> **If your headings render unstyled, you skipped step 4.** `@murad/tokens` bundles no
> mode on purpose — a default would get silently accepted instead of decided.

### Dark mode

Core ships both themes. Toggle by putting `.dark` on `<html>`. The `@custom-variant` line
above is what makes Tailwind's `dark:` prefix follow that class instead of the OS setting.
Omit it and `dark:` utilities will disagree with the tokens.

---

## STEP 3 — The `@source` gotcha

Tailwind v4 only generates utilities for class names it can *see* in scanned files.
Anything shipped inside `node_modules` is invisible by default.

Token imports work without this. **Shipped components do not.** If you also install
`@murad/react`, add:

```css
@source "../node_modules/@murad/react/dist";
```

Failure mode to recognise: components render with correct structure and no styling, while
the config looks entirely fine. That is always this.

---

## STEP 4 — Verify before calling it done

- [ ] Exactly one mode is imported — not zero, not two.
- [ ] `@source` is declared if any component package is installed.
- [ ] **No raw hex, `rgb()`, or Tailwind palette names** (`zinc-800`, `blue-500`) appear in
      component code. Every colour goes through a semantic token.
- [ ] Both themes checked. Not just the one you develop in.
- [ ] Every interactive element has a visible `:focus-visible` state.
- [ ] In `product` mode: no interactive target below 44px without `.tap-expand`.
- [ ] Accent-coloured **text** uses `--color-accent-text`, never `--color-accent`.
- [ ] Text sitting **on** an accent fill is `--color-ink`, not white and not canvas.
- [ ] Reduced-motion honoured on anything animated.

---

## The accent has a contrast ceiling

The coral is a mid-lightness colour, which makes it a good fill and a bad text colour.
Measured, not estimated — ratios below are from the actual `oklch()` values:

| Pair | Ratio | Body (4.5) | Large / UI (3.0) |
|---|---|---|---|
| `accent` as text on canvas | 3.58:1 | ❌ | ✅ |
| **`accent-text` as text on canvas** | **4.79:1** | ✅ | ✅ |
| white text on accent fill | 3.74:1 | ❌ | ✅ |
| canvas text on accent fill | 3.58:1 | ❌ | ✅ |
| **`ink` text on accent fill** | **4.76:1** | ✅ | ✅ |
| `ink-muted` on canvas | 4.65:1 | ✅ | ✅ |
| `accent` as text on dark canvas | 4.76:1 | ✅ | ✅ |

Two rules fall out of this, and they are not interchangeable:

**Text ON an accent fill → `--color-ink`.** A coral button takes dark ink, not white.
This looks unusual next to the reflex of white-on-coloured-button, but white is 3.74:1
and fails. Ink passes at 4.76:1.

**Accent AS text → `--color-accent-text`.** This token resolves to the darker coral in
light mode and the standard coral in dark mode, because the relationship inverts between
themes. Using `--color-accent` for a body-size link is a WCAG AA failure in light mode.

`--color-accent` remains correct for: fills, borders, focus rings, underlines, dividers,
and display-size text at 24px or above.

---

## Hard rules

These are not preferences. Violating them is what makes the system stop looking like itself.

**Radius.** Only `--radius-sharp` (0) or `--radius-diecut` (1.5rem). Never a value between.
A 4px or 8px radius is the single fastest way to make this look like every other Tailwind
site. Die-cut is reserved for meta-layers — callouts, modals, feature cards — never buttons,
inputs, or images.

**Colour.** Reference tokens, never primitives. `bg-surface`, not `bg-white`. `text-ink-muted`,
not `text-gray-500`. If a needed colour has no token, that is a signal to add one to core
deliberately — not to inline a hex.

**Accent scarcity.** Coral marks the primary action and nothing else. A page with four coral
elements has no primary action. Semantic feedback uses `--color-success` / `--color-error`.

**Accent is a fill, not a text colour.** `--color-accent` for fills, borders, focus rings and
display type; `--color-accent-text` for anything below 24px. See the contrast table above.

**Type in `product` mode.** No Playfair. It is absent from that mode's font stack for a
reason: below ~24px its high stroke contrast collapses, and dense UI is nothing but small
type. `--font-serif` resolves to the sans stack there so ported components degrade
gracefully rather than falling back to Times.

**Motion.** Compositor-only properties — `transform`, `opacity`, `filter`. Never animate
`width`, `height`, `top`, or `left`. Every animation needs a `prefers-reduced-motion` exit.

**Focus.** Never `outline: none` without an equivalent replacement. The core focus ring
exists so no project can quietly ship an invisible one.

---

## Escape hatches

Legitimate deviations, with the conditions that make them legitimate.

**A serif hero inside a `product` app.** Import the Playfair face directly in that one
route's stylesheet and apply it at 32px or larger only. Do not raise it to a mode-level
default — the moment it appears in a table header the reason for the split is gone.

**A third mode.** Justified when a real project fits neither — a print-oriented view, a
marketing microsite with a distinct density. Add `src/modes/<name>.css`, export it from
`package.json`, and document its decision row in STEP 1. Do not fork `core.css`.

**A per-project accent.** Almost always wrong; it discards the strongest identity signal in
the package. If a client project genuinely requires its own brand colour, rebind
`--murad-accent` in the consuming project's `:root` *after* importing core. Treat this as a
white-label exception, not as customisation.

---

## Extending the system

Adding a token is a design decision, so it has a procedure:

1. **Is it identity or expression?** Colour, radius, shadow, motion constant → `core.css`.
   Size, leading, spacing, measure, density → the relevant mode file.
2. **Register in `@theme` pointing at a `--murad-*` primitive.** Never
   `--color-x: var(--color-x)`; that reads as circular and breaks silently on Tailwind
   upgrades.
3. **Define the primitive in both `:root` and `.dark`.** A token that exists in only one
   theme is a bug waiting for a theme toggle.
4. **Write the `@rationale`.** Every block in this package explains why. A token whose
   reason isn't recorded gets deleted by someone in six months — possibly you.

---

## Known issues carried from the source system

Both were found while extracting this package and are **fixed here**. They are still live
in the portfolio, so port the fixes back.

- **`.glass-effect` was invalid CSS.** It used `oklch(var(--color-surface) / 0.6)`, but the
  token is already a complete `oklch()` function — nesting it produces an invalid value that
  fails silently to a transparent background. Fixed with `color-mix()`. (The same bug was
  already found and fixed on the blockquote ornament, per its comment.)

- **Prose links failed WCAG AA in light mode.** `--tw-prose-links: var(--color-accent)`
  measures 3.58:1 against canvas; AA needs 4.5:1. Fixed by introducing `--color-accent-text`
  and pointing prose links and counters at it.

- **Hex comments had drifted from the OKLCH values.** `oklch(0.63 0.16 39)` renders as
  `#d76037`, not the `#e76f51` recorded in the comment; ink is `#17171b`, not `#1c1c1e`.
  Harmless to rendering, but the comments were being used as the basis for contrast
  reasoning, which is how the link issue went unnoticed. Corrected in `core.css`.

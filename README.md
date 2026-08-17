# @murad/tokens

Sharp editorial design identity as a portable package. Framework-agnostic CSS for Tailwind v4.

One visual identity across many projects — **carried by colour, edge, and depth rather than
by typeface**, so a dense dashboard and a long-form essay site can share it without either
one fighting the system.

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
@import "@murad/tokens";
@import "@murad/tokens/product";   /* or /editorial — pick exactly one */
```

## What's in it

| Import | Contents |
|---|---|
| `@murad/tokens` | Identity tokens, interaction engine, effects |
| `@murad/tokens/editorial` | Playfair, 1.75 leading, 65ch measure, breakouts |
| `@murad/tokens/product` | Inter, 1.5 leading, tabular numerals, 44px targets |
| `@murad/tokens/core` | Identity only — colour, radius, shadow, focus |
| `@murad/tokens/engine` | Motion, scroll, transitions |
| `@murad/tokens/effects` | Glass, grain, masks, die-cut |

**Shared by every mode:** the coral accent, ink/canvas/surface semantics, `--radius-sharp: 0`
with the die-cut exception, the shadow language, the focus ring, and all motion.

**Per project:** type scale, leading, measure, density.

## Setup

Read **[DESIGN.md](./DESIGN.md)** — it's the decision flow, not just reference. It covers
mode selection, the load-bearing import order, the `@source` gotcha, a pre-ship checklist,
and the hard rules.

Projects using Claude Code pick up `.claude/skills/murad-design/` automatically, which
enforces the same rules during UI work.

## Design principles

**Tokens, never primitives.** No raw hex or Tailwind palette names in component code.

**Two radii, nothing between.** `0` or `1.5rem`. A 4px radius is the fastest way to make
this look generic.

**Scarce accent.** Coral marks the primary action and nothing else.

**Identity ≠ typeface.** Playfair is an editorial choice, not a brand requirement. It is
deliberately absent from `product` mode.

## Status

`0.1.0` — extracted from [muradmadi.com](https://muradmadi.com). Not yet published to a
registry; install from git.

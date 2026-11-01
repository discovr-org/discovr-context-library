---
name: discovr-code
description: >-
  discovr TypeScript and React code quality bar. Use when writing, editing, or
  reviewing components, hooks, utils, styles, or Next.js app code.
---

# discovr code quality

Highest-quality bar for a dark-only, mobile-first Next.js TypeScript app.

Also read before implementing: `discovr-structure` (paths), `discovr-brand` (tokens), `discovr-interface` (states/layout), `discovr-voice` (strings), `discovr-principles` (product calls).

## Defaults

- TypeScript strict. No `any`. Prefer `unknown` + narrow.
- Server Components by default. Add `"use client"` only for state, effects, or browser APIs.
- Named exports for components and utils. Default export only for Next.js `page` / `layout` / `loading` / `error` files.
- Colocate types with the module. Export a type only when another module needs it.
- Pure utils: no React imports in `utils/` unless the file is a hook (`useX.ts`).

## Components

Every interactive control ships **six states**: default, hover, focus-visible, pressed, disabled, loading.

- `focus-visible`: `focus-ring`, 2px + 2px offset — never remove.
- Hover is desktop-only; never the only way to reveal something.
- Pressed must be visible on touch.
- Disabled: `opacity-disabled` on the whole control, not a grey fill.
- Loading on a button: keep width, swap label for spinner.

Every screen ships **four states**: populated, empty, loading, error. Add offline when the screen can open with no signal.

- Loading content → skeletons at real dimensions (`surface-sunken`), not spinners.
- Spinners → user-triggered actions only.
- Real elements: `<button>`, `<a href>`, `<input>` + `<label>`. No clickable `div`s.
- `aria-label` on every icon-only control. Min tap 44×44, targets ≥ `space-2` apart.

## One-per-screen hard caps

At most one of each in view: `brand-primary` fill, AppBar, Hero, BookingBar, TabBar, Toast.

## Styling

- Tokens only — see `discovr-brand` / `tokens.md`. No raw hex, no magic `44px` / `180ms` / `z-index: 50` literals; use size / motion / zIndex families.
- Styles live in `styles/` as typed helpers or CSS modules co-owned by the feature — not scattered inline hex.
- Prefer CSS variables from `:root` over duplicating values in JS.
- `dvh` not `vh`. Honour `prefers-reduced-motion` (opacity-only fallback).

## Data and money

- First price shown = charged price. Fees and taxes included from the feed card onward.
- `PriceBreakdown` total must match `BookingBar` — keep that invariant testable.
- Spots-left only when real and under five. No fake scarcity, timers, or "N people viewing".
- Money errors: what happened → whether money moved → what to do next.

## Images and perf

- `width` + `height` or `aspect-ratio` on every `<img>`.
- `photo-lift` on every photo (menu-document pages excepted).
- AVIF/WebP, 2× max; lazy below fold; `fetchpriority="high"` on hero only.
- Self-host font subsets. Preload only above-the-fold weights.

## Copy in code

Strings follow `discovr-product`. Sentence case in source. No emoji in the UI. Buttons name the outcome (`Book 6:30pm`), not the mechanism (`Continue`).

## Do / don't

| Do | Don't |
| --- | --- |
| Thin `app/` pages that render a feature entry | Dump UI into `page.tsx` |
| Feature → `shared` imports only | Feature → feature imports |
| Skeleton for content wait | Spinner for content wait |
| Token CSS variables | Raw `#061412` in a component |
| Verdict line on every listing card | Catalogue card with no view |

## Before marking done

1. File sits in the right `components|utils|styles` folder.
2. No raw hex / magic timing / magic z-index.
3. Interactive: six states. Screen: four states (+ offline if needed).
4. One primary bone action in view.
5. Images reserved + `photo-lift`.
6. Keyboard order matches visual order; focus ring visible.

---
name: discovr-brand
description: >-
  discovr brand colours, tokens, type, logo, photo, and motion rules. Use when
  styling UI, picking colours, fonts, icons, images, or writing CSS/tokens.
---

# discovr brand

Dark-only. Mobile-first. Photography is most of every screen. One theme — no `prefers-color-scheme` branching, no `[data-theme]`.

## Colour jobs

| Job | Token use |
| --- | --- |
| Primary action | `brand-primary` fill, `brand-primary-on` label — **one per screen** |
| Secondary action | `line-strong` border, `ink-primary` label, no fill |
| Accent / urgency | `accent-pink` fill on `accent-pink-tint`, or as text |
| Saved / selected | `brand-tint` fill, `ink-primary` label |
| Everything else | `surface-*`, `ink-*`, `line-*` (~85% of any screen) |

Proportion: one bone element and at most one pink element in view.

Hard rules:

- Errors use `state-critical`, never `accent-pink`.
- Dark labels on `brand-primary` and `accent-pink` — never white text on either.
- Body text is `ink-primary` (`#eaebe7`), never pure white.
- Cards stay neutral. Colour lives in the photo, not on the card chrome.
- Raw hex in components is a bug. Tokens only. Full table: [tokens.md](tokens.md).

## Surfaces

`surface-canvas` (`#061412`) is the only surface with hue. Raised/sunken/overlay stay near-neutral so warm photos do not fight green trays.

## Type

| Role | Family | Notes |
| --- | --- | --- |
| Display + logo | Bricolage Grotesque | lowercase or sentence case; never all caps. Negative tracking. |
| UI + body | Instrument Sans | 13–16px sweet spot. Slight positive tracking on label/caption only. |

Self-host both (Latin subset, woff2, `font-display: swap`). Caption is the only caps style — applied by CSS, never typed in strings.

Prices use the `price` style with `tabular-nums`. Max two type styles in one card. Body measure 60–70 characters; gutter `space-4`.

## Space / radius / elevation

4px base. Card padding `space-4`, feed gap `space-5`, section gap `space-6`, two-up gap `space-3`.

`radius-pill` on every button and chip.

Prefer surface change or `line-subtle` over shadow. Reserve `shadow-raised` / `shadow-overlay` for things that float.

## Icons

Lucide only. 24px, 2px stroke, round caps/joins. Inline the icons used. `ink-primary` or `ink-muted` only. Never decorate with bone/pink. Min tap 44×44. An icon alone is never a label.

## Photography

- Real guest photos over stock. No stock where a user can see it.
- Every image: `photo-lift`, fixed aspect-ratio, fixed radius, `width`/`height` set.
- AVIF/WebP, 2× max. `loading="lazy"` below fold; `fetchpriority="high"` on hero.
- No text directly on a photo — bottom scrim from `surface-canvas` at 70% to transparent.
- Missing photo → `surface-sunken` placeholder + category glyph.

## Logo

Full wordmark only. For clear space, icon cuts, and asset files, read `discovr-logo` (and its `reference.md`).

## Motion

Enter 180ms `cubic-bezier(0.2, 0, 0, 1)`. Leave 120ms linear. Opacity and transform only. Honour `prefers-reduced-motion`.

## Light surfaces (outside the app theme)

Email, print, and OG/search sit in light contexts. Email: `#061412` on `#ffffff`, teal wordmark, pink `#c2185b`. Print: invert tickets to dark-on-white. OG can stay dark.

## Web chrome

- `color-scheme: dark` on `:root` **and** `<meta name="color-scheme" content="dark">`
- `theme-color` = `surface-canvas`
- Use `dvh`, not `vh`
- `viewport-fit=cover` + `env(safe-area-inset-*)`
- Handle `:-webkit-autofill` explicitly

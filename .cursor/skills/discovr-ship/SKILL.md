---
name: discovr-ship
description: >-
  discovr MVP ship checklist: tokens, core components, PWA, fonts, perf, a11y,
  browser and legal gates. Use when scaffolding, shipping, reviewing launch
  readiness, PWA, icons, or accessibility.
---

# discovr ship checklist

## Before you call it shippable

1. Read [reference.md](reference.md).
2. Walk the sections that touch this change (tokens / components / PWA / fonts / perf / a11y).
3. Done when every item for that section is true — not "later".

## Priority build order

1. Tokens on `:root` — one theme, `color-scheme: dark` twice (CSS + meta), no raw hex.
2. Six components with six states each: Button → ExperienceCard → Chip → Sheet → Input → Badge.
3. PWA: manifest, any + maskable icons (separate files), apple-touch 180, favicon SVG+32, SW (shell + tickets offline), install prompt (Android now; iOS A2HS after first booking).
4. Light surfaces designed on purpose: email, print, OG.
5. Fonts self-hosted Latin woff2; logo independent of font load.
6. Perf budget: AVIF/WebP 2×, reserved image ratios, lazy below fold, hero `fetchpriority="high"`.
7. A11y: 4.5:1 text, 44×44 taps, focus-ring, keyboard order, 200% text, no colour-only meaning.
8. Real Safari iPhone + mid-range Android Chrome, including offline ticket read.

## Leave until after launch

Light theme, custom typeface, illustration system, motion/haptics spec, brand PDF.

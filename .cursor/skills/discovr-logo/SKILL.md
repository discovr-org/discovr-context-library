---
name: discovr-logo
description: >-
  discovr wordmark and app icon rules. Use when adding logo, favicon, splash,
  manifest icons, OG images, email header, or any branded mark.
---

# discovr logo

## Before you ship a mark

1. Read [reference.md](reference.md).
2. Confirm: full name only — never a monogram, letter, or abbreviation.
3. Pick the colourway: bone on dark, teal on light.
4. Done when clear space, min width, and the correct asset file are all met.

## Hard rules

- Wordmark = outlined Bricolage Grotesque, all lowercase `discovr`, never live text when an outlined file exists.
- Aspect ratio fixed. Never stretch, rotate, recolour outside the two colourways, or add a tagline.
- Clear space = half the wordmark height. Minimum width **80px**. Below that: use nothing.
- App icon: bone wordmark on full-bleed `surface-canvas`. **Two files** — `any` (88% width) and `maskable` (76% width). Do not use one file for both.
- Inside the app: wordmark at most once per screen, usually not at all. Screen titles use `display-md`.
- Favicon 16px will read as texture — accepted cost of the all-name rule. Prefer SVG favicon + 32px PNG.

## Assets

| Use | File |
| --- | --- |
| Dark grounds | `discovr-wordmark-bone.svg` |
| Light grounds (email/print) | `discovr-wordmark-teal.svg` |
| Manifest / apple / favicon | `discovr-app-icon` (purpose any) |
| Android maskable | `discovr-app-icon-maskable` (separate) |

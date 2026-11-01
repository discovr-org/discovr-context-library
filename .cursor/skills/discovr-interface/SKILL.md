---
name: discovr-interface
description: >-
  discovr interface rules for screens, layout, components, states, dark-only
  web, booking UI, and anti-patterns. Use when designing or building any screen
  or component.
---

# discovr interface

## Before you build a screen or component

1. Read [reference.md](reference.md) for the section you touch (layout, detail, components, web, a11y, booking).
2. Apply the five that matter most (below).
3. Draw all four screen states (+ offline if needed) and six interactive states.
4. Done when the ship-gate checklist at the end of the reference passes.

## Five that matter most

1. **One primary action** — `brand-primary` once per screen.
2. **Uniform geometry** — fixed aspect, radius, gap. Photos share order that way.
3. **Every photo** — `photo-lift` + reserved ratio (menu pages excepted).
4. **85% neutral** — bone and pink are events, not surfaces.
5. **Nothing light by accident** — `color-scheme: dark`, autofill handled, `dvh`, safe-area, `theme-color`.

## One-per-screen

One each: brand-primary fill, AppBar, Hero, BookingBar, TabBar, Toast.

## Choosing pairs (quick)

| Need | Pick |
| --- | --- |
| Many-of-many | Chip |
| One-of-few | SegmentedControl |
| Already happened | Toast |
| Ongoing condition | Banner |
| Decision required | Dialog |
| In-task dismissible | Sheet |
| Waiting to read | Skeleton |
| Waiting on an action | Spinner |

## Anti-patterns (do the opposite)

Two bone buttons. Colour on photo cards. Full-bleed photos in a feed. Text on image with no scrim. Icon-only tabs. Carousel as primary browse. Modal onboarding before content. Spinner for content. Pure white type. `100vh`. Off-scale spacing.

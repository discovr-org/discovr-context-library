# discovr design tokens

Ban raw hex in product code. If a colour is not a token, it is a bug. Same for magic numbers that belong to size, motion, or z-index families.

## Colour

| Token | Hex |
| --- | --- |
| `surface-canvas` | `#061412` |
| `surface-raised` | `#141a19` |
| `surface-sunken` | `#1d2422` |
| `surface-overlay` | `#222927` |
| `surface-inverse` | `#f4f0e3` |
| `ink-primary` | `#eaebe7` |
| `ink-muted` | `#9ba3a0` |
| `ink-subtle` | `#808886` |
| `ink-inverse` | `#061412` |
| `line-subtle` | `#283130` |
| `line-strong` | `#636d6b` |
| `brand-primary` | `#f4f0e3` |
| `brand-primary-hover` | `#fffdf4` |
| `brand-primary-press` | `#ded9c9` |
| `brand-primary-on` | `#061412` |
| `brand-tint` | `#22211d` |
| `accent-pink` | `#ff5c9b` |
| `accent-pink-tint` | `#3a1425` |
| `state-positive` | `#57d3a0` |
| `state-positive-tint` | `#0f2a20` |
| `state-notice` | `#f0b74a` |
| `state-notice-tint` | `#2e230c` |
| `state-critical` | `#ff7a63` |
| `state-critical-tint` | `#33150f` |
| `focus-ring` | `#f4f0e3` |

## Light-context overrides (email / print only)

| Use | Value |
| --- | --- |
| Email body text | `#061412` on `#ffffff` |
| Email / print pink | `#c2185b` |
| Email logo | `discovr-wordmark-teal.svg` |

## Space (4px base)

| Token | px |
| --- | --- |
| `space-1` | 4 |
| `space-2` | 8 |
| `space-3` | 12 |
| `space-4` | 16 |
| `space-5` | 20 |
| `space-6` | 24 |

Common: gutter `space-4`, feed gap `space-5`, section gap `space-6`, two-up gap `space-3`.

## Radius / tap / motion

| Token | Value |
| --- | --- |
| `radius-pill` | full pill (buttons, chips) |
| min tap target | 44×44 |
| enter | 180ms `cubic-bezier(0.2, 0, 0, 1)` |
| leave | 120ms linear |

## CSS wiring

Export as custom properties on `:root`. One theme. Set `color-scheme: dark`.

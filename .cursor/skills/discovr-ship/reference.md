# Ship checklist

What to produce from this system before the MVP goes live, in rough order. Written for a dark-only, mobile-first web app.

## Wire the tokens into CSS

- Export `tokens.json` as CSS custom properties on `:root`. The system serves a compiled `tokens.css` you can consume directly.
- **One theme.** No `prefers-color-scheme` branching, no `[data-theme]` switching. That simplification is the main thing dark-only buys you, so do not build the machinery anyway "for later".
- Set `color-scheme: dark` on `:root` **and** `<meta name="color-scheme" content="dark">`. Without both, native controls render light.
- Ban raw hex values in code review. If a colour is not a token, it is a bug.

## Build the six components

Specs and guidelines are on each component's page. In priority order:

- **Button**, primary and secondary and ghost
- **ExperienceCard**, both densities
- **Chip**, with the `brand-tint` selected state
- **Sheet**, with the focus trap
- **Input**, at 16px with autofill handled
- **Badge**, urgency and status

All six interaction states each, including a visible pressed state for touch.

## Web app setup

No App Store or Play Store listing. Installing to the home screen is an optional browser feature, not a separate app.

Every icon below is the full wordmark. Nothing in this product ever ships a single-letter mark.

| Item | Spec |
| --- | --- |
| `manifest.webmanifest` | `name` "discovr", `short_name` "discovr", `display: standalone`, `background_color` and `theme_color` both `#061412`, `orientation: portrait` |
| Manifest icons, `purpose: "any"` | 192 and 512 PNG from `discovr-app-icon.svg` |
| Manifest icon, `purpose: "maskable"` | 512 PNG from `discovr-app-icon-maskable.svg`. **A separate file, not the same one.** Android crops maskable icons to the inner 80% circle; the standard artwork sets the wordmark at 88% of the square and would lose its first and last letters |
| `apple-touch-icon` | 180 by 180 PNG from `discovr-app-icon.svg`. iOS ignores manifest icons entirely |
| iOS meta | `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style: black-translucent` |
| Viewport | `width=device-width, initial-scale=1, viewport-fit=cover` |
| Favicon | `discovr-app-icon.svg` as the SVG favicon, plus a 32 by 32 PNG fallback. The 16px fallback will be a texture rather than a readable word; that is the accepted cost of the all-name rule |
| Splash | The bone wordmark centred on `#061412` at 60% of screen width. iOS needs static splash images per device size |
| Service worker | App shell precached; confirmed bookings and tickets cached for offline |
| Install prompt | `beforeinstallprompt` on Android and desktop. iOS needs a designed "Add to Home Screen" instruction, shown after a first successful booking |

**Test the icons before launch**, not after. Install to a real iPhone home screen and a real Android one, check the maskable crop in Chrome DevTools' Application panel, and look at the favicon against a white browser tab bar. The wordmark is tight at these sizes by design, so verify rather than assume.

## Light surfaces

Named in the brand book, still undesigned. Do not discover these in production.

- **Email templates** for booking confirmation and receipt, built light: `#061412` on `#ffffff`, `discovr-wordmark-teal.svg`, pink as `#c2185b`.
- **`@media print` stylesheet** that inverts tickets to dark-on-white and hides navigation. People print tickets for guides.
- **OG image** at 1200 by 630 on the teal-black ground with the bone wordmark, checked inside a light chat app.

## Fonts

- Subset Bricolage Grotesque and Instrument Sans to Latin, serve `woff2`, self-hosted.
- Preload only the weights above the fold.
- The logo does **not** depend on this. The wordmark ships as outlined paths, so a font that fails to load cannot break the brand. Body and display type will fall back; the logo will not.

## Performance, which is a design deliverable here

Your users are on mobile data with patchy coverage. Set a budget before you build, not after.

- AVIF or WebP, 2x maximum, `loading="lazy"` below the fold, `fetchpriority="high"` on the hero.
- `width` and `height` or `aspect-ratio` on every image. This is your Cumulative Layout Shift score.
- Inline critical CSS.
- Test on a throttled 4G profile and a mid-range Android, not your laptop.

## Accessibility pass before launch

- Every text and background pair at 4.5:1, or 3:1 at 24px and above. One theme, so one pass.
- Every tap target at least 44 by 44.
- `focus-ring` visible on every interactive element with its 2px offset.
- Keyboard reachable in visual order.
- Tested at 200% text and 400% browser zoom.
- No meaning carried by colour alone.
- Tested at minimum screen brightness in a dark room, and at maximum in direct sun.

## Browser testing

- Real Safari on a real iPhone, in the tab and installed to the home screen.
- Chrome on a mid-range Android, same.
- Autofill on a payment form in both.
- The return journey from a UPI or bank redirect.
- Offline: open the app with data off and confirm a booking is still readable.

## Legal and admin

- Trademark search on the name in India, in the travel and booking class. This matters more than usual now that the logo is the name itself, with no symbol to fall back on.
- Font licences. Bricolage Grotesque and Instrument Sans are both SIL Open Font License, which permits outlining the wordmark and self-hosting. Record the versions you ship.
- Lucide is MIT licensed. Keep the attribution file.
- Image rights for every host photo. Get a written licence in the host onboarding flow, not after.

## What to leave until after launch

- A light theme.
- A custom typeface.
- An illustration system.
- A motion and haptics specification.
- Brand guidelines as a PDF. This system is the guidelines.

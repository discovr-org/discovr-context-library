# Living system context

Update this file when durable facts change. Do not put secrets here.

## Repos

| Repo | Role | Default port |
| --- | --- | --- |
| `discovr-frontend` | Guest PWA | 3000 |
| `discovr-admin` | Internal catalog dashboard | 3001 |
| `discovr-backend` | Fastify API (`api/`) | 8080 |
| `discovr-context-library` | Skills + agent rules (this repo) | — |

## Backend modules (`discovr-backend/api/src`)

| Module | Owns |
| --- | --- |
| `catalog` | Experiences, hosts, feed, listing photos / menu pages / story images, avatar uploads |
| `communication` | SMS via Twilio (`SmsSender` → `TwilioSmsSender`) |
| `user` | Guest users by `phone_e164` |
| `auth` | OTP request/verify, opaque `sessionId`, `/v1/auth/*` |

HTTP talks only to module facades (`CatalogService`, `AuthService`, `UserService`, `CommunicationService`).

## Auth / SMS (facts)

- OTP SMS **From** is alphanumeric sender `DSCOVR` (`TWILIO_FROM_NUMBER=DSCOVR`), not the support phone.
- Support / customer phone (`+918169992510`) is **not** the Twilio From for OTP.
- OTP copy shape: `{code} is your discovr code. It works for 10 minutes. Do not share it with anyone.`
- Session: opaque token returned as `sessionId`; client sends `Authorization: Bearer <sessionId>`.
- Tables: `users`, `auth_otps`, `auth_sessions` (migration `007_auth_users_sessions.sql`), `user_saves` (migration `008_user_saves.sql`).
- Guest saves live in `localStorage` (`discovr.saved`) until login; after OTP verify the PWA merges local + remote and `PUT /v1/me/saves`. Also `GET/PUT/DELETE /v1/me/saves` (+ `PUT/DELETE /v1/me/saves/:slug`).
- Sign-up name step: `POST /v1/auth/otp/verify` returns `isNewUser` (true while `users.name` is empty, so drop-offs get asked again) and `name`. `PATCH /v1/me` `{ name }` saves the first name (max 40). Migration `013_user_name.sql`.
- List with us: `POST /v1/host-applications` `{ name, phone, city, description }` (no account) → `host_applications` (migration `014_host_applications.sql`). Lives in the `user` module next to trip-update signups.
- Help page: `POST /v1/help-requests` `{ category, message, name, phone }` (no account; category is one of `problem_with_experience`, `host_unreachable`, `felt_unsafe`, `price_or_payment_issue_with_host`, `something_else`) → `help_requests` (migration `015_help_requests.sql`).
- Save heart and Profile tab open signup when signed out; the heart still writes localStorage immediately.
- Local override: `AUTH_DATA_SOURCE=memory` when Supabase auth tables are missing; catalog can stay on supabase.

## Images

- Admin edit listing manages every listing image: **Photos** (carousel; one is the cover), **Menu pages** (label per page), and **Story photos** (replace a `photo` block in `content.blocks`, saved with "Save changes"). Routes in skill `discovr-backend` reference, "Listing images".
- Home cards use the cover, else the first carousel photo. `ExperienceCard` swaps to the fallback image if a photo fails to load.
- Nov 2026: `media.discovr.in` has no DNS record, so old-site photos (most listings, incl. menus and story photos) don't load. Re-upload through the admin (goes to `cdn.discovr.in`) or restore that host.
- Admin, listing, host avatar and story uploads go to Cloudflare R2 bucket `assets`, served at `https://cdn.discovr.in/images/<uuid>.<ext>` (`R2ImageUploader`, used when `CLOUDFLARE_ACCOUNT_ID` + `R2_ACCESS_KEY_ID` + `R2_SECRET_ACCESS_KEY` + `R2_BUCKET` + `R2_PUBLIC_URL` are all set). Falls back to Cloudflare Images (`CLOUDFLARE_TOKEN`). Old-site photos stay on `media.discovr.in` — do not repoint it.

## Env names (no values)

Backend: `SUPABASE_*`, `CATALOG_DATA_SOURCE`, `AUTH_DATA_SOURCE`, `CLOUDFLARE_*`, `R2_*`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER`, `CORS_ORIGIN`, `API_PORT`.

Frontend / admin: `NEXT_PUBLIC_API_BASE_URL` (no trailing slash). Local `http://127.0.0.1:8080`; production = Render API HTTPS origin. Must be present at Next **build** time.

Guest app feed, search, saved, and experience detail load only from `/v1/catalog/*` (no frontend seed/demo listings). Detail maps API `content` (FAQs, meeting point, links) and listing `hours`; cover photo fills empty `photos`. `public/demo/` assets may still be referenced by cover URLs in the DB until real uploads replace them.

## Guest home

- Big wordmark + "Goa, from people who know where to go", then search. Past the search bar a top bar slides in with a small wordmark and the chips (Tonight, This weekend, Near me, Under ₹1,000, Food and drink, Nightlife, Wellness).
- The frontend owns the home feed: `HOME_SECTIONS` in `discovr-frontend/src/features/home/utils/homeSections.ts` holds each section's title, layout (`rail` = tall swipe cards with name, area, heart; `stack` = one big card each with photo, title, verdict line, area · duration, "From ₹X", heart), listing slugs and order. Sections with no live listings are hidden. There is no `feed_sections` table.
- `GET /v1/catalog/cards` returns every published listing with an active host as a flat card list (no layout, no sections). Migration `010_listing_format.sql` only fills `content.format`.
- APIs send raw values, not display text: `durationMinutes` (cards and detail) and no fallback photo. The frontend formats duration (`shared/utils/formatDuration.ts`) and picks the fallback image.

## Guest detail

- Place vs experience: `content.format` (`venue` / `trip`), else tags. Full section order lives in skill `discovr-interface` reference ("The detail screen").
- Place: name only, area + open/closed, quote, 4 actions, About (italic + photo), Price, Menu, disclaimer + links, More in Goa (2 columns), bar "Ask for the price" + "Call to book".
- Experience: "by {host}", price / area / open today / duration + group, quote, 4 actions, About (italic pitch + description + photo), Details, included / not included boxes, The process, Health advisory, Things to get with you, Know before you go, FAQs, Run by card, disclaimer + links, More in Goa, bar with "or book on {host}" when `content.links.book` is set.
- Bars: BookingBar + TabBar always on; AppBar turns slim/solid past the photo and never hides.
- Listing About story: `experiences.content.blocks` (`StoryBlock[]`: heading, line, quote, text, photo, forNotFor, steps, callout, checklist, faq), rendered by `StoryBlocks`; listings without blocks get one built from their fields (`features/experience/utils/story.ts`). Old-site stories, photos and menus loaded by migration `012_story_blocks.sql`. Venues show "by …" and the host section only when the host credit is `owned`.
- Footer (`SiteFooter`, home + detail): links, `POST /v1/trip-updates` WhatsApp signup (user module, table `trip_update_signups`, migration `011_trip_updates.sql`), legal line, faded wordmark.
- Phone normalising lives in `api/src/shared/phone.ts` (auth + user share it).
- New `content` keys (admin edit form): `walkIn` (venues), `pitch`, `fitnessLevel`, `instructorRatio`, `healthAdvisory`, `carry[]`, `knowBeforeYouGo[]`. Admin save now keeps content keys the form doesn't edit.

## Cards cache

- `GET /v1/catalog/cards` is cached in API memory for 24 hours (`ListCards`). Concurrent misses share one Supabase read.
- Any admin write (listing or host create / edit / delete, listing photo add / reorder / delete, avatar upload) clears it, so edits show on the next request.
- Manual: `POST /v1/admin/cache/cards/refresh` clears and re-caches; admin Dashboard has a "Refresh home feed" button.
- Cache is per process: a Render restart or cold start starts empty and pays one database read.

## Legal pages

- `/legal/privacy`, `/legal/terms`, `/legal/disclaimer` share `features/legal/components/LegalPage.tsx`. Old `/privacy` and `/terms` redirect (308) in `next.config.ts`.
- Privacy has an analytics on/off switch (`ConsentSettings`) stored on the device for 12 months (`features/legal/utils/analyticsConsent.ts`). PostHog is not installed yet; when it is, it must check `isAnalyticsOn()` before loading.
- Page chrome: `AppShell` renders `SiteFooter` on every screen (`footer={false}` to hide, `footerSource` for the signup source; defaults to the path). Text pages (About, How we pick, List with us, Help, `/legal/*`) live in `src/app/(info)/` and share `InfoShell` (nav, tab bar, gutters, footer).

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
| `catalog` | Experiences, hosts, feed, cover/avatar uploads |
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
- Save heart and Profile tab open signup when signed out; the heart still writes localStorage immediately.
- Local override: `AUTH_DATA_SOURCE=memory` when Supabase auth tables are missing; catalog can stay on supabase.

## Images

- Admin cover / host avatar uploads go through Cloudflare Images (`CLOUDFLARE_TOKEN`). Prefer Images API token over Global `cfk_` key.

## Env names (no values)

Backend: `SUPABASE_*`, `CATALOG_DATA_SOURCE`, `AUTH_DATA_SOURCE`, `CLOUDFLARE_*`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER`, `CORS_ORIGIN`, `API_PORT`.

Frontend / admin: `NEXT_PUBLIC_API_BASE_URL` (no trailing slash). Local `http://127.0.0.1:8080`; production = Render API HTTPS origin. Must be present at Next **build** time.

Guest app feed, search, saved, and experience detail load only from `/v1/catalog/*` (no frontend seed/demo listings). Detail maps API `content` (FAQs, meeting point, links) and listing `hours`; cover photo fills empty `photos`. `public/demo/` assets may still be referenced by cover URLs in the DB until real uploads replace them.

## Guest home

- Big wordmark + "Goa, from people who know where to go", search, then chips. Past the search bar the chip row sticks to the top with a small wordmark.
- Place rows (sundowners, nightlife, OG local bars, cocktails, eat like a Goan local) are portrait swipe rails: name, area, heart. Experience rows (surfing, wakeboarding, padel, mudbath, tattoo, pottery) are one big landscape card: photo, title, verdict line, area · duration, "From ₹X", heart. Kayaking is a portrait rail with prices (`feed_sections.show_price`).
- Migration `010_feed_layout.sql` sets titles, density, order, `show_price`, folds mangrove kayaking into one Kayaking rail, and fills `content.format` (venue for place rails, trip for the rest). The API reads `show_price` but falls back if 010 isn't applied.

## Guest detail

- Place vs experience: `content.format` (`venue` / `trip`), else tags. Full section order lives in skill `discovr-interface` reference ("The detail screen").
- Place: name only, area + open/closed, quote, 4 actions, About (italic + photo), Price, Menu, disclaimer + links, More in Goa (2 columns), bar "Ask for the price" + "Call to book".
- Experience: "by {host}", price / area / open today / duration + group, quote, 4 actions, About (italic pitch + description + photo), Details, included / not included boxes, The process, Health advisory, Things to get with you, Know before you go, FAQs, Run by card, disclaimer + links, More in Goa, bar with "or book on {host}" when `content.links.book` is set.
- Bars: BookingBar + TabBar always on; AppBar turns slim/solid past the photo and never hides.
- Footer (`SiteFooter`, home + detail): links, `POST /v1/trip-updates` WhatsApp signup (user module, table `trip_update_signups`, migration `011_trip_updates.sql`), legal line, faded wordmark.
- Phone normalising lives in `api/src/shared/phone.ts` (auth + user share it).
- New `content` keys (admin edit form): `walkIn` (venues), `pitch`, `fitnessLevel`, `instructorRatio`, `healthAdvisory`, `carry[]`, `knowBeforeYouGo[]`. Admin save now keeps content keys the form doesn't edit.

## Feed cache

- `GET /v1/catalog/feed` is cached in API memory for 24 hours (`ListFeed`). Concurrent misses share one Supabase read.
- Any admin write (listing or host create / edit / delete, cover or avatar upload) clears it, so edits show on the next request.
- Manual: `POST /v1/admin/cache/feed/refresh` clears and re-caches; admin Dashboard has a "Refresh home feed" button.
- Cache is per process: a Render restart or cold start starts empty and pays one database read.

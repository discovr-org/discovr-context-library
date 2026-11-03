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
- The only image host is `cdn.discovr.in` (R2 bucket `assets`). `media.discovr.in` was never ours and has no DNS record. Migration `012_story_blocks.sql` points ~160 old-site photos (carousel, menus, story) at `media.discovr.in/images/<id>`; those files are not in the bucket, so they don't load until the originals are put in R2 and the URLs rewritten, or they're re-uploaded through the admin.
- Admin, listing, host avatar and story uploads go to Cloudflare R2 bucket `assets`, served at `https://cdn.discovr.in/images/<uuid>.<ext>` (`R2ImageUploader`, used when `CLOUDFLARE_ACCOUNT_ID` + `R2_ACCESS_KEY_ID` + `R2_SECRET_ACCESS_KEY` + `R2_BUCKET` + `R2_PUBLIC_URL` are all set). Falls back to Cloudflare Images (`CLOUDFLARE_TOKEN`).

## Env names (no values)

Backend: `SUPABASE_*`, `CATALOG_DATA_SOURCE`, `AUTH_DATA_SOURCE`, `CLOUDFLARE_*`, `R2_*`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER`, `CORS_ORIGIN`, `API_PORT`.

Frontend / admin: `NEXT_PUBLIC_API_BASE_URL` (no trailing slash). Local `http://127.0.0.1:8080`; production = Render API HTTPS origin. Must be present at Next **build** time.

Guest app analytics: `NEXT_PUBLIC_POSTHOG_KEY` (project "discovr", EU; empty = analytics off), `NEXT_PUBLIC_POSTHOG_HOST=/ingest`. `app_version` reads Vercel's `NEXT_PUBLIC_VERCEL_GIT_COMMIT_SHA`.

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
- Booking: `content.bookVia` = `call` (default; button calls the host phone) or `link` (button "Book on {host}" opens `content.links.book.url`, venues too). Admin "Booking" section has the toggle. Link `label`s are legacy; the app shows icons, so the admin only edits URLs.
- Story photos: `content.hideStoryPhotos = true` makes the app drop every `photo` block from the story (stored or auto-built). Admin "Story photos" section has the "Show photos in the story" switch; turning it back on removes the key.
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

## Analytics (PostHog, guest app only)

- Boot: `src/instrumentation-client.ts` calls `bootAnalytics()` (`shared/analytics/utils/posthog.ts`). Loads only with a key and analytics on. EU, `person_profiles: identified_only`, history-change pageviews, pageleave, autocapture, exceptions. Replays mask every input plus anything with `data-ph-mask` (phone on Profile and in the sign-in sheet).
- Proxy: `next.config.ts` rewrites `/ingest/static|array/*` to `eu-assets.i.posthog.com` and `/ingest/*` to `eu.i.posthog.com` (`skipTrailingSlashRedirect: true`).
- Every event gets `page_type` (home, experience, saved, search, profile, info), `app_surface` (pwa / browser), `app_version`.
- Identity: `AuthProvider` calls `identifyUser(userId)` on sign-in and for returning sessions, `resetUser()` on sign-out. Internal id only.
- Rules: one typed list, `shared/analytics/utils/events.ts` (`EventMap`, `object_action` snake_case). Send only through `track(event, props)`. No name, phone or email fields: the `NoPii` type fails the build if one is added. Free text the user typed (help message, report note) is never sent; only fixed options (`category`, `kind`) and the search query.
- Cards: wrap a list in `<TrackList list section… layout>` and pass `track={cardTrack(item, index)}` to `ExperienceCard`. Card events carry slug, title, format, price_inr, place, list (home / search / saved / more_in_goa), section_id/title, position (1-based), layout, search_query. A card click stores its source in sessionStorage so `experience_viewed` reports `source_list / source_section / source_position`.
- Views: `useTrackView` (50% on screen, or filling half the screen, for 1 s; zero-height markers count when on screen) for cards, home sections, story start/end. `useScrollDepth` sends 25/50/75/100 on home and listings.
- Events: discovery `experience_card_viewed`, `experience_card_clicked`, `home_section_viewed`, `search_performed` (600 ms after typing stops). Listing `experience_viewed`, `story_viewed`, `story_completed`, `faq_opened`, `hero_photo_changed`, `menu_opened`, `menu_page_viewed`, `hours_expanded`, `directions_clicked`, `call_clicked`, `share_clicked`, `report_opened`, `report_submitted`, `scroll_depth`. Conversion `book_clicked` (main success event), `experience_saved` / `experience_unsaved` (`from`). Sign-in `auth_sheet_opened`, `otp_requested`, `otp_verified`, `otp_failed`, `name_submitted`, `auth_sheet_closed`, `signed_out`. Nav `tab_clicked`, `contact_sheet_opened`, `contact_clicked`, `footer_link_clicked`, `help_request_submitted`. Plus `analytics_opted_out`.
- Shared components stay analytics-free where a feature owns the context: they take optional callbacks (`BookingBar.onBookClick`, `ActionRow.onAction`, `InfoList.onAction`, `HeroGallery.onIndexChange`, `MenuViewer.onPageChange`, `StoryBlocks.onFaqOpen`). Book buttons and tabs carry `data-attr`.
- Dev: `ph.debug()` logs every event in the console. React Strict Mode sends `experience_viewed` twice in dev only.

## Legal pages

- `/legal/privacy`, `/legal/terms`, `/legal/disclaimer` share `features/legal/components/LegalPage.tsx`. Old `/privacy` and `/terms` redirect (308) in `next.config.ts`.
- Privacy has an analytics on/off switch (`ConsentSettings`) stored on the device for 12 months (`shared/analytics/utils/analyticsConsent.ts`). Turning it off sends `analytics_opted_out`, stops replays and opts PostHog out; turning it on loads or resumes PostHog.
- Page chrome: `AppShell` renders `SiteFooter` on every screen (`footer={false}` to hide, `footerSource` for the signup source; defaults to the path). Text pages (About, How we pick, List with us, Help, `/legal/*`) live in `src/app/(info)/` and share `InfoShell` (nav, tab bar, gutters, footer).

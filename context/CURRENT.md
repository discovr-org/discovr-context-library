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
| `assistant` | MCP server (`POST /mcp`) so ops can draft hosts and listings from their own Claude / ChatGPT |

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

Guest app analytics: `NEXT_PUBLIC_POSTHOG_KEY` (project "new-discovr", US; empty = analytics off), `NEXT_PUBLIC_POSTHOG_HOST=/ingest`. `app_version` reads Vercel's `NEXT_PUBLIC_VERCEL_GIT_COMMIT_SHA`.

Guest app feed, search, saved, and experience detail load only from `/v1/catalog/*` (no frontend seed/demo listings). Detail maps API `content` (FAQs, meeting point, links) and listing `hours`; cover photo fills empty `photos`. `public/demo/` assets may still be referenced by cover URLs in the DB until real uploads replace them.

## Guest home

- Big wordmark + "Goa, from people who know where to go", then search. Past the search bar a top bar slides in with a small wordmark and the chips (Tonight, This weekend, Near me, Under ₹1,000, Food and drink, Nightlife, Wellness).
- The frontend owns the home feed: `HOME_SECTIONS` in `discovr-frontend/src/features/home/utils/homeSections.ts` holds each section's title, layout (`rail` = tall swipe cards with name, area, heart; `stack` = one big card each with photo, title, verdict line, area · duration, "From ₹X", heart), listing slugs and order. Sections with no live listings are hidden. There is no `feed_sections` table.
- `GET /v1/catalog/cards` returns every published listing with an active host as a flat card list (no layout, no sections). Migration `010_listing_format.sql` only fills `content.format`.
- APIs send raw values, not display text: `durationMinutes` (cards and detail) and no fallback photo. The frontend formats duration (`shared/utils/formatDuration.ts`) and picks the fallback image.

## Guest detail

- Place vs experience: `content.format` (`venue` / `trip`), else tags. Full section order lives in skill `discovr-interface` reference ("The detail screen").
- Place: name only, area + open/closed, quote, 4 actions, About built from the admin fields (quote, story photos, then Price (note + unit), Best time, Time to spend, Age, Group size, Address, Also known as, Know before you go = additional info + tips, FAQs), Menu (pages, or a "See the menu" link from `content.links.menu`), "Terms and policy" toggle (policy summary + `termsDetail`, booking, cancellation), disclaimer + links, More in Goa, bar "From ₹X" + short `priceUnit` (≤ 20 chars) or "Ask for the price" when no price is set. Rule: anything filled in the admin shows on the page. Instagram: the place's `content.links.instagram` beats the host's handle.
- Experience: "by {host}", price / area / open today / duration + group, quote, 4 actions, About (italic pitch + description + photo), Details, included / not included boxes, The process, Health advisory, Things to get with you, Know before you go, FAQs, Run by card, disclaimer + links, More in Goa, bar with "or book on {host}" when `content.links.book` is set.
- Booking: `content.bookVia` = `call` (default; button calls the host phone) or `link` (button "Book on {host}" opens `content.links.book.url`, venues too). Admin "Booking" section has the toggle. Both book buttons and the action row's Call need login: guests get the sign-in sheet (`requireAuth("call" | "book")`) and the call or host page follows on success. After login the host page tries a new tab and falls back to the same tab if the browser blocks it. Link `label`s are legacy; the app shows icons, so the admin only edits URLs.
- Story photos: `content.hideStoryPhotos = true` makes the app drop every `photo` block from the story (stored or auto-built). Admin "Story photos" section has the "Show photos in the story" switch; turning it back on removes the key.
- Bars: BookingBar + TabBar always on; AppBar turns slim/solid past the photo and never hides.
- Listing About story: `experiences.content.blocks` (`StoryBlock[]`: heading, line, quote, text, photo, forNotFor, steps, callout, checklist, faq), rendered by `StoryBlocks`; listings without blocks get one built from their fields (`features/experience/utils/story.ts`). Places always build from fields (stored blocks only lend their photos), so admin edits show within the 60 s page cache. Old-site stories, photos and menus loaded by migration `012_story_blocks.sql`. Venues show "by …" and the host section only when the host credit is `owned`.
- Footer (`SiteFooter`, home + detail): links, `POST /v1/trip-updates` WhatsApp signup (user module, table `trip_update_signups`, migration `011_trip_updates.sql`), legal line, faded wordmark.
- Phone normalising lives in `api/src/shared/phone.ts` (auth + user share it).
- Hours (`experiences.schedule`): `timeSlot` (usual hours), `closedDays[]` and `altDays[]` + `altTimeSlot` ("Mon" … "Sun", admin day pickers in "When"). Free-text `days` still works ("Closed Tuesday.", "Tuesdays off"). The app shows times as typed ranges ("8am – 1am", never "midnight", no "Open from"); best time lives in About, not the hours row.
- New `content` keys (admin edit form): `walkIn` (venues), `pitch`, `fitnessLevel`, `instructorRatio`, `healthAdvisory`, `carry[]`, `knowBeforeYouGo[]`. Admin save now keeps content keys the form doesn't edit.

## Admin dashboard (`discovr-admin`)

- Pages: Dashboard (`/`), Listings, Hosts, Search, edit / add listing and host.
- Dashboard "Needs attention" lists non-archived listings with gaps: no verdict, no photo, no price, or a published listing whose host is inactive. Photo check uses `hasPhoto` on `GET /v1/catalog/experiences` (cover URL or any carousel photo).
- Guest "List with us" and Help submissions are not shown in the admin (a Requests inbox was built and removed on request). They sit in `host_applications` and `help_requests` (migrations `014`, `015`).
- Data loading: one shared copy per list, `shared/utils/useCatalog.ts`. Listings and hosts load on their own (a page never waits for the list it does not use); in-flight requests are shared, shows the saved copy at once and refreshes in the background. Shell search, Dashboard, Listings, Hosts and Search all use it. After a write, call `reload()`.
- Change first, save after (optimistic): set active/inactive, delete listing, host edit / active / delete update the screen at once via `patchExperienceLocal` / `removeExperienceLocal` / `patchHostLocal` / `removeHostLocal` (`useCatalog.ts`), then call the API. On failure the helper's `undo` puts that one row back and a toast says why. A load that started before a local change is dropped so it can't overwrite it. No refetch after success. Rule: new admin actions on listed rows follow this pattern. Not done this way yet: photo uploads, add listing / host, full edit-listing save (the table is patched after the save returns).
- Instant first paint: the last copy of each list is kept in `localStorage` (`discovr.admin.experiences`, `discovr.admin.hosts`), read through `useSyncExternalStore` (server render sees null, so no hydration mismatch). A reload shows the saved data at once and refreshes it in the background. Every local change is saved there too.
- Keyboard: `⌘K` search (runs on the local copy), `c` new listing, `g` then `o` / `l` / `h` / `s` goes to Overview / Listings / Hosts / Search. Ignored while typing in a field. Shown as hints on the nav links.
- Speed rules the admin follows (from the "How Linear is so fast" write-up): change the screen first and save after, no spinner for something that will almost always succeed; read from a local copy; animate only `transform`, `opacity` and colours, 80–160 ms, no list transitions, reduced motion respected; fonts via `next/font` (one preloaded Latin file each).
- Not done this way: add listing / host and photo uploads (the server makes the id), render-only-the-changed-row (lists are ~25 rows, not worth it), service-worker precache.
- Add listing makes a draft with the basics, then opens Edit, which holds every field the guest story page shows (pitch, About, price note, hours, included / not included, health advisory, things to bring, know before you go, FAQs, links, menu, policy, photos). Edit shows a "Finish setting up" card while the listing lacks a photo, verdict, About, price, hours or FAQs; each item jumps to its section.
- Edit listing: `dirty` flag warns on tab close (`beforeunload`), jump bar pinned under the top bar (measured offsets in `--jump-top` / `--jump-offset`) that links to each section (`#sec-<name>`) and highlights the section you are in (`IntersectionObserver`, last tab at page bottom), sticky save bar says "Unsaved changes" or "All changes saved".
- **No login.** `/v1/admin/*` is open to anyone who has the API URL. An email-OTP login (`@discovr.in` only, Cloudflare or Resend email) was built and then removed on request. Add it back before the API URL is shared widely.
- Not done yet: sortable listing columns, filters kept in the URL, auto-refresh of the home feed after publish (today it is the Dashboard "Refresh home feed" button).

## Cards cache

- `GET /v1/catalog/cards` is cached in API memory for 24 hours (`ListCards`). Concurrent misses share one Supabase read.
- Any admin write (listing or host create / edit / delete, listing photo add / reorder / delete, avatar upload) clears it, so edits show on the next request.
- Manual: `POST /v1/admin/cache/cards/refresh` clears and re-caches (this instance only); admin Dashboard has a "Refresh home feed" button.
- TTL is 60 s (was 24 h) so several API instances can only disagree for a minute.
- Cache is per process: a Render restart or cold start starts empty and pays one database read.

## Scaling guards (API)

- Rate limits (`@fastify/rate-limit`, per IP, in-process counters, `trustProxy: true`): default 300 / min; `POST /v1/auth/otp/request` 5 / 10 min; `/v1/auth/otp/verify` 15 / 10 min; `/v1/trip-updates` 5 / 10 min; `/v1/host-applications` 3 / 10 min; `/v1/help-requests` 5 / 10 min. Over the cap: `429 { error: { code: "rate_limited" } }`. Helper `limit(max)` in `http/rate-limits.ts`. With N instances the real cap is N times higher; add a shared store if abuse shows up.
- Sessions: `SessionCache` keeps token hash → user id for 60 s (max 10k). Routes that only need the id (`/v1/me/*`) call `AuthService.userIdFor`; `/v1/auth/me` still loads the user. Sign-out clears the entry on that instance; others keep it up to 60 s.
- Migration `017_purge_auth.sql` schedules a nightly `pg_cron` job that deletes expired OTPs and old sessions (needs the `pg_cron` extension).
- Admin lists (`GET /v1/catalog/experiences` and `/hosts`) use `StaleCache`: fresh 20 s, then served instantly while refreshing in the background up to 5 min, then wait. Any admin write drops them and reloads at once. Measured Nov 2026: Supabase answers in 0.15–0.5 s but the same list via the Render API took 1.5–2.2 s, so check that Render and Supabase share a region.
- Not done: Cache-Control / CDN caching of `/v1/catalog/*`, admin login, paginated cards, monitoring.

## Analytics (PostHog, guest app only)

- Boot: `src/instrumentation-client.ts` calls `bootAnalytics()` (`shared/analytics/utils/posthog.ts`). Loads only with a key and analytics on. US cloud, `person_profiles: identified_only`, history-change pageviews, pageleave, autocapture, exceptions. Replays mask every input plus anything with `data-ph-mask` (phone on Profile and in the sign-in sheet).
- Proxy: `next.config.ts` rewrites `/ingest/static|array/*` to `us-assets.i.posthog.com` and `/ingest/*` to `us.i.posthog.com` (`skipTrailingSlashRedirect: true`).
- Every event gets `page_type` (home, experience, saved, search, profile, info), `app_surface` (pwa / browser), `app_version`.
- Identity: `AuthProvider` calls `identifyUser(userId)` on sign-in and for returning sessions, `resetUser()` on sign-out. Sends the internal id plus person properties `name`, `phone_number` (E.164) and `created_at` (`$set_once`, from the verify response's `user.createdAt`; sessions saved before this get it on next sign-in). Never email. The privacy page still says analytics never gets name or phone; owner chose to leave it. Events themselves still carry no name (`NoPii`).
- Rules: one typed list, `shared/analytics/utils/events.ts` (`EventMap`, `object_action` snake_case). Send only through `track(event, props)`. No name, phone or email fields: the `NoPii` type fails the build if one is added. Free text the user typed (help message, report note) is never sent; only fixed options (`category`, `kind`) and the search query.
- Cards: wrap a list in `<TrackList list section… layout>` and pass `track={cardTrack(item, index)}` to `ExperienceCard`. Card events carry slug, title, format, price_inr, place, list (home / search / saved / more_in_goa), section_id/title, position (1-based), layout, search_query. A card click stores its source in sessionStorage so `experience_viewed` reports `source_list / source_section / source_section_title / source_position`. The card heart sends `experience_saved` / `experience_unsaved` itself with the same card fields and `location: "card"`.
- Context on every event: listing-page events spread `listingProps(x)` (slug, title, format, area, price_inr, ask_for_price). Button events add `cta` (stable id, e.g. `call_to_book`, `book_on_site`, `share`, `directions`), `cta_label` (words on the button) and `location` (`card`, `app_bar`, `action_row`, `info_list`, `booking_bar`, `menu_section`, `story`, `listing_links`, `tab_bar`, `top_nav`, `contact_sheet`, `footer`). `tab_clicked` and `contact_sheet_opened` say `tab_bar` or `top_nav`; `faq_opened` has a 1-based `position`; `menu_opened` has `start_page`.
- Views: `useTrackView` (50% on screen, or filling half the screen, for 1 s; zero-height markers count when on screen) for cards, home sections, story start/end. `useScrollDepth` sends 25/50/75/100 on home and listings.
- Events: discovery `experience_card_viewed`, `experience_card_clicked`, `home_section_viewed`, `search_performed` (600 ms after typing stops). Listing `experience_viewed`, `story_viewed`, `story_completed`, `faq_opened`, `hero_photo_changed`, `menu_opened`, `menu_page_viewed`, `hours_expanded`, `directions_clicked`, `call_clicked`, `share_clicked`, `report_opened`, `report_submitted`, `scroll_depth`. Conversion `book_clicked` (main success event), `experience_saved` / `experience_unsaved` (`location`). Sign-in `auth_sheet_opened`, `otp_requested`, `otp_verified`, `otp_failed`, `name_submitted`, `auth_sheet_closed`, `signed_out`. Nav `tab_clicked`, `contact_sheet_opened`, `contact_clicked`, `footer_link_clicked`, `help_request_submitted`. Plus `analytics_opted_out`, `qr_scanned`, and `location_captured`.
- Device location: `useDeviceLocation` runs once in `Providers`. No bar of our own. After the first real scroll (48px; ignores the iOS toolbar jump), if analytics is on and this device has no saved fix, it calls `getCurrentPosition`, which opens the browser's own prompt. At most one prompt per visit (`sessionStorage` `discovr.locationAsked`). A fix is saved in `localStorage` (`discovr.deviceLocation`) and not asked again. A browser deny is remembered. Already-granted permission refreshes quietly. The read asks for GPS (`enableHighAccuracy`), falling back to a coarser read on timeout. PostHog gets `device_latitude`, `device_longitude`, `device_location_accuracy_m`, `location_source: device` on every event (added in `before_send` from the saved fix, so even the first pageview has it) and on the person once they're signed in (`identified_only` drops it before that). `$geoip_*` is still the IP guess and is often wrong in Safari. Filter on `device_latitude`.
- T-shirt QR: `https://discovr.in/q/t` (and `/q/t/`) 307s to `/?utm_source=tshirt&utm_medium=qr&utm_campaign=tshirt` (`next.config.ts`, temporary). On boot, PostHog stores those utm tags for the visit, sends `qr_scanned`, then the query is removed so a shared homepage link is not another scan. After sign-in, person property `qr_campaign` = `tshirt` is set once (`$set_once`). Anonymous visits still count via the event and `utm_campaign` (no person profile until identify). Filter any event where `utm_campaign = tshirt`, or count `qr_scanned`.
- Shared components stay analytics-free where a feature owns the context: they take optional callbacks (`BookingBar.onBookClick`, `ActionRow.onAction`, `InfoList.onAction`, `HeroGallery.onIndexChange`, `MenuViewer.onPageChange`, `StoryBlocks.onFaqOpen`). Book buttons and tabs carry `data-attr`.
- Dev: `ph.debug()` logs every event in the console. React Strict Mode sends `experience_viewed` twice in dev only.

## Legal pages

- `/legal/privacy`, `/legal/terms`, `/legal/disclaimer` share `features/legal/components/LegalPage.tsx`. Old `/privacy` and `/terms` redirect (308) in `next.config.ts`.
- Privacy has an analytics on/off switch (`ConsentSettings`) stored on the device for 12 months (`shared/analytics/utils/analyticsConsent.ts`). Turning it off sends `analytics_opted_out`, stops replays and opts PostHog out; turning it on loads or resumes PostHog.
- Page chrome: `AppShell` renders `SiteFooter` on every screen (`footer={false}` to hide, `footerSource` for the signup source; defaults to the path). Text pages (About, How we pick, List with us, Help, `/legal/*`) live in `src/app/(info)/` and share `InfoShell` (nav, tab bar, gutters, footer).

## Assistant (MCP)

- Module `discovr-backend/api/src/assistant` (facade `AssistantService`, tools in `application/tools.ts`, input shapes in `domain/listing-input.ts`). Route: `http/mcp-routes.ts`. Stateless Streamable HTTP, SDK `@modelcontextprotocol/sdk`.
- Tools. Read: `find_hosts`, `list_listings`, `get_listing`, `needs_attention` (same gaps as the admin Dashboard, rule copied server-side), `find_place`. Draft: `create_host_draft` (host `active: false`), `create_listing_draft` (`status: "draft"`, create then patch). Edit: `update_listing` (merges `content` / `schedule` into what is stored because the repo replaces them wholesale; lists replace), `update_host`. Gated, marked `destructiveHint` so clients ask first: `set_listing_status`, `set_host_active`. No delete, no slug / host change. Drafts are the only create path, the AI cannot publish unless asked through `set_listing_status` (the use-case still needs a verdict). Text only: chat photos can't reach tools, so `get_photo_upload_link` returns the edit page `#sec-photos` (listing) or `/hosts` (avatar) for a human to upload.
- `find_place`: port `PlaceFinder`, adapter `WebPlaceFinder`. Map link (reads `!3d/!4d`, `@lat,lng`, `?q=`, follows short links on Google hosts only) or OpenStreetMap Nominatim (India only, thin on small shacks, ~1 req/s). Swap in Google Places behind the port if it misses.
- Field guide: `application/field-guide.ts` is sent as the MCP server `instructions` and as prompt `new_listing`. Voice rules mirror skill `discovr-voice`; update both together.
- The AI uses ops' own subscription, so we pay no tokens.
- **OAuth is a stub**: `/.well-known/*`, `/oauth/register|authorize|token` auto-approve, `/mcp` accepts any bearer token. Anyone with the API URL can use it. Replace with real admin login before sharing the URL.
- Env: `ADMIN_URL` (optional, default `https://discovr-admin.vercel.app`) builds the review links the tools return.
- Check: `cd discovr-backend/api && npx tsx scripts/check-assistant.ts`.
- Not done: photo tools that accept files (ChatGPT `openai/fileParams` could skip the upload link), `ai_draft` flag, tool-call log, real login, timeline / process steps (the guest app hardcodes "The Process").

## Trip end prompt ("Till when are you in Goa?")

- `discovr-frontend/src/features/trip`: `TripEndGate` (mounted in `Providers`) shows the non-dismissible `TripEndPrompt` once per visit after 30 s of use (tab visible, touched or scrolled in the last 15 s), if there is no answer or `askAgainOn` has come. It waits while the sign-in sheet is open.
- Signed in: the API is the source of truth (`GET /v1/auth/me` → `user.trip`; `PUT /v1/me/trip`). Guests: answer lives in `localStorage` (`discovr.trip`) and moves up to the DB on sign-in if the DB has none.
- PostHog person gets `trip_status` and `trip_last_day`; events `trip_end_shown`, `trip_end_answered`.
- Device location events also carry `device_city/state/country`, looked up once via BigDataCloud (`api-bdc.io`, spot rounded to ~1 km). Privacy page names both.

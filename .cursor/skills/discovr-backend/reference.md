# discovr backend reference

## Request flow

```
PWA (Next.js) → api :8080 /v1/catalog/...
                     ↓
              http/routes (thin)
                     ↓
           CatalogService (module API)
                     ↓
              use-cases (in-process)
                     ↓
           ExperienceRepository port
                     ↓
          memory seed  |  Supabase (discovr-catalog)
```

## Port

| Process | Default | Env |
| --- | --- | --- |
| api | 8080 | `API_PORT` |

Frontend: `NEXT_PUBLIC_API_BASE_URL=http://localhost:8080`

## Catalog schema (`discovr-catalog`)

See migration: `discovr-backend/api/supabase/migrations/001_init.sql`.

### Core tables

- `hosts` — person who picks up; phone, IG, verified, languages
- `experiences` — listing SOE; money as int INR; schedule/policy jsonb; rating aggregates; cover denorm for feed
- `experience_photos` / `experience_inclusions` / `experience_menu_pages` — ordered children
- `experience_tags` — filter tags
- `reviews` — raw reviews; aggregates denormalised on experiences

## Folder map

```
discovr-backend/api/src/
  main.ts
  app.ts
  config.ts
  http/routes.ts
  http/auth-routes.ts
  shared/errors.ts
  catalog/                   # listings
  communication/             # SMS (Twilio)
  user/                      # guest users by phone + user_saves
  auth/                      # OTP + sessions
```

## Catalog list extras

- `GET /v1/catalog/experiences` rows also carry `hasPhoto` (cover URL or any carousel photo).

## Listing images (catalog, admin)

- Tables: `experience_photos` (carousel, `sort_order`, `is_cover`) and `experience_menu_pages` (`sort_order`, `label`). Port `ExperienceMediaRepository` (rows only); cover rules live in use-cases.
- The cover is one of the carousel photos (`is_cover`), copied into `experiences.cover_photo_url` / `cover_photo_alt` so cards and search read one column. Deleting the cover promotes the first remaining photo.
- `GET /v1/admin/experiences/:id` returns `photos[]` (`id, src, alt, isCover`) and `menuPages[]` (`id, src, label, alt`).
- `POST /v1/admin/experiences/:id/photos` (multipart `file`, optional `alt`, `cover=true`) adds at the end. `PUT …/photos` `{ photos: [{ id, alt }], coverId }` sends the full list in order. `DELETE …/photos/:photoId`.
- `POST /v1/admin/experiences/:id/cover` = add photo + set as cover (kept for old clients).
- `POST|PUT|DELETE /v1/admin/experiences/:id/menu-pages[/:pageId]`, same shape with `label`.
- `POST /v1/admin/images` (multipart) returns `{ url }` for story `photo` blocks; the admin saves the URL into `content.blocks` with the listing.
- Multipart text fields must come before the file. JPG/PNG/WebP/GIF, under 10MB (`application/image-file.ts`).
- Cards: `imageSrc` = `cover_photo_url`, else the `is_cover` photo, else the first photo.

## User saves

- Table: `user_saves` (`user_id`, `experience_slug`) — migration `008_user_saves.sql`.
- Routes (Bearer session): `GET/PUT /v1/me/saves`, `PUT/DELETE /v1/me/saves/:slug`.
- `PATCH /v1/me` `{ name }` (Bearer) sets first name; OTP verify returns `isNewUser` = no name yet.
- `PUT /v1/me/trip` `{ status: date|later|local|not_yet, lastDay? }` (Bearer) saves "Till when are you in Goa?" (migration `018_trip_end.sql`: `trip_status`, `trip_last_day`, `trip_ask_again_on`). Server sets `askAgainOn` = last day (or answer day for not_yet) + 3 days; `local` = never. Every user payload (`/v1/auth/me`, verify, `PATCH /v1/me`) carries `trip` (null until answered).
- PWA login handoff (migration `019_auth_handoffs.sql`): `POST /v1/auth/handoff` (Bearer) → `{ code }` (32 random bytes, hashed at rest, 30 min, one use); `POST /v1/auth/handoff/exchange` `{ code }` (public, 10 per 10 min per IP) → `{ sessionId, user, isNewUser }`, a brand new 30-day session. Consume is one atomic UPDATE. iOS Home Screen apps start with empty storage; this moves a signed-in browser's login into the app.
- Admin login: `POST /v1/admin/login` `{ email, password }` → `{ token, expiresAt }` (12 h). Every other `/v1/admin/*` route needs `Authorization: Bearer <token>` (hook in `http/admin-routes.ts`, registered first). Email `admin@discovr.in`; the password is a constant in `auth/application/admin-auth.ts`, stored only as its SHA-256. Tokens are HMAC-signed with `SUPABASE_SECRET_KEY`. Public routes (`/v1/catalog/*`, `/v1/me/*`, auth) are not gated. The `/mcp` route is separate and still open.
- WhatsApp via BotSpace (`BOTSPACE_API_KEY`, `BOTSPACE_CHANNEL_ID`; migration `020_message_sends.sql`): templates live in `communication/domain/templates.ts` (`id` = BotSpace templateId, `body` with {{1}}.., `variables` = names from `firstName | lastDay | daysLeft`). Routes: `GET /v1/admin/message-templates`, `GET /v1/admin/users?signedUp=today|7d|all&leavingWithinDays=0..60&templateId=` (phones hidden; sends go by id), `POST /v1/admin/messages/send { templateId, userIds, allowRepeat }` (max 200, 4 at once, skips people who already got that template unless `allowRepeat`), `GET /v1/admin/messages/:batchId?refresh=1` (BotSpace delivery status). The BotSpace key travels in the query string and is scrubbed from errors. "Today" is India time (`shared/ist.ts`).
- Welcome WhatsApp (migration `027_welcome_jobs.sql`, env `BOTSPACE_WELCOME_TEMPLATE_ID`): `VerifyOtp` queues one `welcome_jobs` row for a brand-new user (`UserService.findOrCreateByPhone` returns `{ user, created }`); `WelcomeQueue` (communication) polls every 30 s, sends the template with `{{1}}` = first name or "there", retries every 5 min up to 3 more times, logs to `message_sends`. `run_at` is a lease, so a crash just retries; claiming is per-row compare-and-set, safe with two API copies. No template id = nothing queued.
- `POST /v1/host-applications` (public) stores the List with us form in `host_applications`.
- `POST /v1/help-requests` (public) stores the Help form in `help_requests`.
- Guest PWA keeps slugs in `localStorage` until login, then syncs via `PUT /v1/me/saves`.


## Splitting later

1. Extract `catalog/` into its own package/service.
2. Change `http/routes` to call HTTP instead of in-process use-cases (or keep a BFF).
3. Give catalog its own port/deploy. Public browser origin can stay the API/BFF.

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
- `POST /v1/host-applications` (public) stores the List with us form in `host_applications`.
- `POST /v1/help-requests` (public) stores the Help form in `help_requests`.
- Guest PWA keeps slugs in `localStorage` until login, then syncs via `PUT /v1/me/saves`.


## Splitting later

1. Extract `catalog/` into its own package/service.
2. Change `http/routes` to call HTTP instead of in-process use-cases (or keep a BFF).
3. Give catalog its own port/deploy. Public browser origin can stay the API/BFF.

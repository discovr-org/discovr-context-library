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

## User saves

- Table: `user_saves` (`user_id`, `experience_slug`) — migration `008_user_saves.sql`.
- Routes (Bearer session): `GET/PUT /v1/me/saves`, `PUT/DELETE /v1/me/saves/:slug`.
- `PATCH /v1/me` `{ name }` (Bearer) sets first name; OTP verify returns `isNewUser` = no name yet.
- `POST /v1/host-applications` (public) stores the List with us form in `host_applications`.
- Guest PWA keeps slugs in `localStorage` until login, then syncs via `PUT /v1/me/saves`.


## Splitting later

1. Extract `catalog/` into its own package/service.
2. Change `http/routes` to call HTTP instead of in-process use-cases (or keep a BFF).
3. Give catalog its own port/deploy. Public browser origin can stay the API/BFF.

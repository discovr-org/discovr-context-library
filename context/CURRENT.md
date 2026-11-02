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

---
name: discovr-backend
description: >-
  discovr backend modular monolith, SOLID, catalog module boundaries, Supabase
  schema, and API shape. Use when creating or changing anything under
  discovr-backend/, SQL migrations, /v1 routes, catalog domain logic, or
  frontend API clients.
---

# discovr backend

Read [reference.md](reference.md) for schema, folder map, and request flow.

## Always true

1. **One deployable API process** (`discovr-backend/api`, port `8080`). The browser talks only to this origin — never to Supabase with a service key, never to a second Node port for catalog.
2. **Modules stay separated by folder.** `catalog/` owns listings, hosts, feed, cover/avatar uploads. `communication/` owns SMS. `user/` owns guest users. `auth/` owns OTP + sessions. `http/` only routes and maps status codes. Future domains (e.g. `booking/`) get their own folder inside the same process until scale forces a split.
3. **HTTP talks to module APIs only.** Each domain exposes one facade (e.g. `CatalogService`). Routes never import use-cases, repos, or Supabase. Other modules call the same facade — never each other’s internals.
4. **SOLID is mandatory** — see below. Do not invent a parallel pattern.
5. **Money is integers** (INR rupees). Never float.
6. **Published listings always have a verdict.** Drafts may omit; API never returns published without one.
7. **Ports over clients.** Domain and use-cases depend on interfaces; Supabase is an adapter.

## SOLID (do not skip)

| Letter | In this repo |
| --- | --- |
| **S** | One module does one job. A route handler does not query Supabase. A repository does not format HTTP. A use-case does not know Fastify. |
| **O** | Extend with new adapters / use-cases. Do not fork `if (db === "supabase")` trees inside domain code. |
| **L** | Any `ExperienceRepository` implementation must honour the same contract (including empty / not-found behaviour). |
| **I** | Small ports. Do not force feed code to depend on a fat "God" repository that also writes reviews. |
| **D** | Use-cases receive ports via constructor / factory. Composition root (`main`) wires adapters. |

### Layer order (catalog module)

```
http/routes → catalog/api (CatalogService facade)
                    ↓
           catalog/application (use-cases) → catalog/domain
                    ↓
             ports (interfaces)
                    ↓
   catalog/infrastructure (supabase, memory, presenters)
```

Import direction: **inward only**. Domain never imports Fastify, Supabase, or fetch. `http/` imports only `catalog/api` (+ shared error types). `app.ts` stays thin (Fastify + plugins + register routes).

## Layout

```
discovr-backend/
  api/                 # the only runnable backend
    src/
      main.ts          # composition root — wires repos into module APIs
      app.ts           # Fastify shell only
      http/            # public /v1 routes → module APIs
      catalog/         # CatalogService
      communication/   # CommunicationService (Twilio SMS)
      user/            # UserService
      auth/            # AuthService (OTP + sessionId)
      shared/          # shared errors
    supabase/          # discovr-catalog + auth migrations
```

OTP SMS sender id: `DSCOVR` (`TWILIO_FROM_NUMBER`). See `context/CURRENT.md`.

Do **not** add `discovr-backend/gateway` or `discovr-backend/catalog` as separate Node processes unless/until you intentionally split for scale.

## Stack

- TypeScript strict, Node 20+, Fastify, Zod.
- Supabase Postgres as system of record for catalog (`discovr-catalog` project).
- Prefer **memory adapter** when `CATALOG_DATA_SOURCE=memory` so local UI works without Supabase.
- Server clients via `@supabase/server` / `@supabase/server/core` (`createAdminClient`). Env: `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY`, `SUPABASE_JWKS_URL` (see `discovr-backend/.env`). Do **not** use legacy `SUPABASE_SERVICE_ROLE_KEY` / `SUPABASE_ANON_KEY`. Load skill `supabase-server` before changing auth or client wiring.

## API conventions

- Public paths: `/v1/...` on the API.
- JSON only. Errors: `{ "error": { "code": "not_found", "message": "…" } }`.
- List endpoints are cursor- or keyset-ready; do not ship unbounded arrays for production queries.
- Detail by `slug` for shareable URLs; UUID stays internal.

## Schema mindset (Headout-like, Gen Z curated)

- **No layout or feed curation in the backend.** The API serves listings (`GET /v1/catalog/cards`, detail by slug); the frontend decides sections, order and how cards render.
- **Hosts** are first-class. Listings do not embed host columns that drift.
- **Photos / inclusions / menu pages** are child tables (ordered).
- **Tags** power Tonight / weekend / vibe filters.
- **Rating aggregates** live on `experiences` for feed speed; raw reviews are a separate table.
- Index what you filter: `status`, `slug`, tags, geo (later), `search_vector`.

## When to split into real services

Only when a module needs its own deploy cadence, scaling, or team ownership. Until then: one port, many folders.

## Do not

- Put business rules in route handlers.
- Let the frontend hold Supabase service-role keys.
- Run two Node servers for gateway + catalog “for purity”.
- Return float money or bare averages without `rating_count`.
- Add a top-level `discovr-backend/utils` deployable. Shared types stay inside `api` or a future package.

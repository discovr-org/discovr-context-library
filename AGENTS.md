# discovr agent notes

**Context SoT:** this file + `.cursor/` live in [discovr-context-library](https://github.com/discovr-org/discovr-context-library). Agents must stay current with it.

## Loop

1. **Before** any product/backend/admin change: read `.cursor/rules/discovr-build.mdc`, then matching skills; skim `context/CURRENT.md`.
2. **Build** only against those rules — do not invent brand, voice, layout, or API shape.
3. **After** durable changes: update skills / `context/CURRENT.md` / this file (skill `discovr-context`) and push the context library.

Always-on rule: `.cursor/rules/discovr-build.mdc`.

## Skills (`.cursor/skills/`)

| Skill | When |
| --- | --- |
| `discovr-principles` | Product / UX decisions |
| `discovr-interface` | Screens and components |
| `discovr-brand` | Colour, type, tokens, photo, motion |
| `discovr-voice` | Any user-facing copy |
| `discovr-structure` | Where files go |
| `discovr-logo` | Wordmark, icons, splash |
| `discovr-ship` | PWA, a11y, launch checklist |
| `discovr-code` | TypeScript / React quality |
| `discovr-backend` | API modules, Supabase, SOLID |
| `discovr-context` | Load/update this library |
| `supabase-server` | `@supabase/server` auth modes, clients, JWKS |

## Codebases

| Path / repo | Role |
| --- | --- |
| `discovr-frontend/` | Guest PWA (port `3000`) |
| `discovr-admin/` | Internal dashboard (port `3001`) |
| `discovr-backend/api/` | One Fastify process (port `8080`): `catalog`, `communication`, `user`, `auth` |

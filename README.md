# discovr context library

Source of truth for **how agents build discovr**. Skills, always-on rules, and living system notes live here so every agent starts from the same map.

Repos:

- [discovr-frontend](https://github.com/discovr-org/discovr-frontend)
- [discovr-backend](https://github.com/discovr-org/discovr-backend)
- [discovr-admin](https://github.com/discovr-org/discovr-admin)

## Layout

```
AGENTS.md                 # agent entry — read first
.cursor/rules/            # always-on Cursor rules
.cursor/skills/           # product / code / backend skills
.agents/skills/           # shared agent skills (e.g. supabase-server)
context/CURRENT.md        # living facts (repos, env names, module map)
skills-lock.json          # pinned external skill sources
```

## Agent loop (required)

1. **Before building** — read `AGENTS.md`, then the matching skill(s) from the build gate in `.cursor/rules/discovr-build.mdc`.
2. **While building** — follow those skills; do not invent brand, voice, layout, or API shape.
3. **After done** — if you changed product rules, APIs, folder shape, env vars, or durable decisions, update this library in the same session and push to `main` (skill: `discovr-context`).

## Install into a local workspace

From a monorepo root that contains `discovr-frontend` / `discovr-backend` / `discovr-admin`:

```bash
git clone git@github.com:discovr-org/discovr-context-library.git discovr-context-library
# Symlink or copy .cursor + AGENTS.md into the workspace root agents use:
cp -R discovr-context-library/.cursor .
cp discovr-context-library/AGENTS.md .
cp -R discovr-context-library/.agents . 2>/dev/null || true
```

Or keep this repo as the only place agents load skills from (Cursor project skills path).

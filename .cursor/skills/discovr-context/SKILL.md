---
name: discovr-context
description: >-
  Keep discovr-context-library current. Use before non-trivial discovr work to
  load living context, and after finishing work that changes APIs, skills,
  folder shape, env names, or durable product decisions — update and push.
---

# discovr context library

Repo: `https://github.com/discovr-org/discovr-context-library`  
Local path (when cloned beside apps): `discovr-context-library/`

## Before building

1. Read workspace `AGENTS.md` (or this library’s `AGENTS.md`).
2. Read `.cursor/rules/discovr-build.mdc` and open every matching skill.
3. Skim `context/CURRENT.md` for module map, auth/SMS, and env names.

## After done (same session)

Update this library when **any** of these changed:

- New or renamed module / route / table / env var
- New skill, rule, or “always true” product constraint
- Durable decision (e.g. Twilio sender id, session shape, ports)

### Steps

1. Edit the matching skill / `context/CURRENT.md` / `AGENTS.md` / rule.
2. Keep secrets out — names only in `CURRENT.md`.
3. If this library is its own git repo, commit and push `main`:

```bash
cd discovr-context-library
git add -A
git commit -m "Update context: <short why>"
git push origin main
```

4. If the workspace root mirrors `.cursor/` + `AGENTS.md`, copy the same edits there so local agents stay in sync.

## Do not

- Put tokens, keys, or phone OTPs in this repo.
- Leave a new API surface undocumented in `discovr-backend` skill or `CURRENT.md`.

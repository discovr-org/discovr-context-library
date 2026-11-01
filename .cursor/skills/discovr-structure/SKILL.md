---
name: discovr-structure
description: >-
  discovr feature file layout and Next.js placement rules. Use when adding
  screens, components, utils, styles, routes, or deciding where a file lives.
---

# discovr file structure

Every feature is a folder. Inside it, three kinds of files only.

```
[feature]/
  components/[Name].tsx
  utils/[name].ts
  styles/[name].ts
```

## Root map (repo)

```
discovr-frontend/              # Next.js PWA — see map below
discovr-admin/                 # Next.js internal dashboard (port 3001)
discovr-backend/
  api/                         # one process (8080): http/ + catalog/ modules
```

Backend layout and SOLID rules: skill `discovr-backend`.
Admin uses the same `features|shared` shape and brand tokens as the guest app.

## Frontend map (`discovr-frontend/src`)

```
src/
  app/                         # routes + layout only — no business UI here
  features/
    home/
      components/
      utils/
      styles/
    experience/
      components/
      utils/
      styles/
    booking/
      components/
      utils/
      styles/
  shared/
    components/                # Button, Chip, Sheet, Badge, …
    utils/
    styles/                    # tokens, typography, motion, global CSS
  types/                       # shared domain types only
```

## Placement rules

| Thing | Lives in |
| --- | --- |
| Route page / layout / loading / error | `src/app/...` — thin: import from a feature |
| Screen-specific UI | `features/[feature]/components/` |
| Reusable control used by 2+ features | `shared/components/` |
| Feature-only helper | `features/[feature]/utils/` |
| Cross-feature helper | `shared/utils/` |
| Feature CSS-in-JS / style helpers | `features/[feature]/styles/` |
| Design tokens, type scale, motion | `shared/styles/` |

## Naming

- Components: `PascalCase.tsx` — `ExperienceCard.tsx`, `BookingBar.tsx`
- Utils / styles: `camelCase.ts` — `formatPrice.ts`, `photoLift.ts`
- One component per file. Co-locate its types in the same file unless shared.
- Feature folder names: lowercase, singular domain noun (`experience`, `booking`).

## Import direction

```
app → features → shared
```

- Features never import from `app/`.
- Features never import from sibling features. Lift shared bits to `shared/`.
- `shared/` never imports from `features/`.

## Admin map (`discovr-admin/src`)

Same rules as the guest app. Shell chrome lives in `features/shell/`. Screen bodies live in their own feature (`overview`, later `listings`, …). Shared Button/Badge/tokens stay in `shared/`.

```
src/
  app/                 # routes only
  features/
    shell/             # AdminShell, nav
    overview/
  shared/
    components/
    styles/
```

## What stays out of `app/`

No feature components, no domain utils, no token definitions. A page file wires data and renders one feature entry component.

```tsx
// src/app/page.tsx — thin
import { HomeFeed } from "@/features/home/components/HomeFeed";

export default function Page() {
  return <HomeFeed />;
}
```

## Before creating a file

1. Is it route chrome? → `app/`
2. Used by one feature? → that feature's `components|utils|styles`
3. Used by two or more? → `shared/`
4. Never invent a fourth top-level kind (`hooks/`, `lib/`, `helpers/` as siblings). Hooks are utils; put `useX.ts` under `utils/`.

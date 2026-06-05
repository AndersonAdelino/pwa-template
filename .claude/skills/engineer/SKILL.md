---
name: engineer
description: Phase 3 of the PWA pipeline. Acts as Senior Software Engineer — sets up Supabase backend (schema + RLS + auth + types) and implements the Next.js + Tailwind + PWA frontend from the approved design. Use when starting Phase 3, building the app, setting up Supabase, or scaffolding the project. Triggers on "fase 3", "engineer", "implementa", "build", "Next.js", "Supabase".
---

# Skill: engineer — Software Engineer (Phase 3)

Self-contained Phase 3. Two stages: (1) Supabase backend, (2) Next.js frontend.

## Output language

Chat updates and PROJECT.md notes in pt-BR. All code, identifiers, file paths, route names, SQL stay in English.

## Context to load

1. `PROJECT.md` — accumulated decisions, app name
2. `prd.md` — entities, acceptance criteria
3. `wireframes/wireframe.html` — dynamic states, business rules
4. `designs/design.html` — tokens, components, screens, interactions

Phase 2 must be done. If not, instruct user to run `designer` skill first.

---

## STAGE A — Supabase backend

### A.1 Supabase MCP — required tools

| Need | MCP tool |
|------|----------|
| List existing tables | `mcp__supabase__list_tables` |
| Apply migration | `mcp__supabase__apply_migration` |
| Run ad-hoc SQL | `mcp__supabase__execute_sql` |
| Get project URL | `mcp__supabase__get_project_url` |
| Get publishable keys | `mcp__supabase__get_publishable_keys` |
| Generate TS types | `mcp__supabase__generate_typescript_types` |
| Security/perf advisors | `mcp__supabase__get_advisors` |
| Read logs | `mcp__supabase__get_logs` |

Never ask user for Supabase URL, project_ref, or keys — always fetch via MCP.

### A.2 Derive entities

From design + PRD:
- Each "noun" the user creates/edits/deletes → one table
- Fields derived from form inputs and card layouts in `designs/design.html`
- Relationships from navigation hierarchy (list → item = parent-child)
- Constraints from acceptance criteria in `prd.md`

Show mapping in chat for confirmation before applying anything.

### A.3 Inspect current database

```
mcp__supabase__list_tables()
```

If tables already exist, show them and ask whether to drop/modify/keep. Never silently overwrite.

### A.4 Migration template (per entity)

```sql
create table [entity] (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users not null,
  -- derived fields from design forms/cards
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- RLS mandatory on every table
alter table [entity] enable row level security;

create policy "users_own_[entity]" on [entity]
  for all using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

-- updated_at trigger for editable entities
create trigger set_updated_at
  before update on [entity]
  for each row execute function moddatetime(updated_at);
```

If `moddatetime` isn't enabled, prepend `create extension if not exists moddatetime;`.

### A.5 Apply migrations via MCP

For each migration:

```
mcp__supabase__apply_migration(
  name: "00X_<descriptive_name>",
  query: "<SQL content>"
)
```

Also save SQL to `supabase/migrations/00X_<descriptive_name>.sql` for git versioning. Confirm with `list_tables()` after each apply.

### A.6 Auth configuration

Default: email + password sign-up, email confirmation enabled, anonymous sign-in disabled.

If PRD requires social providers or magic links, instruct user to enable in Supabase dashboard (MCP doesn't expose auth config).

### A.7 Generate TypeScript types

```
mcp__supabase__generate_typescript_types()
```

Save to `src/lib/supabase/database.types.ts`. Single source of truth for entity shapes — never write manual types for DB rows.

### A.8 Run advisors

```
mcp__supabase__get_advisors()
```

Fix every CRITICAL and HIGH finding before continuing. Common: missing RLS, public read policies, missing indexes on foreign keys.

### A.9 Output env values

```
url = mcp__supabase__get_project_url()
keys = mcp__supabase__get_publishable_keys()
```

Write to `.env.local` (must be in `.gitignore`):
```
NEXT_PUBLIC_SUPABASE_URL=<url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<keys.anon_key>
```

### A.10 Backend checklist

- [ ] Every table has RLS enabled
- [ ] Every table has at least one policy
- [ ] No table missing `user_id` (unless explicitly shared/public)
- [ ] All foreign keys have indexes
- [ ] `database.types.ts` regenerated
- [ ] `get_advisors` reports zero CRITICAL/HIGH findings
- [ ] `.env.local` written, listed in `.gitignore`

---

## STAGE B — Next.js frontend

### B.1 Scaffold

```bash
npx create-next-app@latest src \
  --typescript --tailwind --eslint --app --src-dir \
  --import-alias "@/*" --no-git
```

Then extend:

```bash
cd src && npm install \
  @supabase/supabase-js @supabase/ssr \
  react-hook-form @hookform/resolvers zod \
  clsx
```

### B.2 Structure

```
src/
  app/
    (auth)/
      login/page.tsx
      signup/page.tsx
      layout.tsx
    (app)/
      layout.tsx          # AppShell with global nav
      [route]/page.tsx    # one route per design screen
    layout.tsx
    globals.css
  components/
    ui/                   # design system: Button, Card, Input, Badge, ...
    layout/               # Header, Nav, AppShell
    features/             # domain components derived from design
  lib/
    supabase/
      client.ts           # createBrowserClient
      server.ts           # createServerClient (RSC + server actions)
      middleware.ts       # auth middleware helper
      database.types.ts   # generated by Stage A
    hooks/                # one hook per business entity
    utils.ts              # cn(), formatters, ...
  middleware.ts           # protects (app)/* routes
  types/
    index.ts
```

### B.3 Design system in Tailwind

From `designs/design.html`:
- Extract every CSS custom property (`--color-*`, `--space-*`, `--radius-*`, `--font-*`)
- Map them in `tailwind.config.ts` under `theme.extend`
- Mirror as CSS variables in `globals.css` for runtime theming

Per reusable component identified in design annotations:
- Implement under `components/ui/` with typed props
- Document variants in component file (e.g. `variant: "ok" | "low" | "critical"`)
- Match every visual state from design (hover, focus, disabled, checked, loading)

### B.4 Supabase client helpers

`src/lib/supabase/client.ts`:
```typescript
import { createBrowserClient } from "@supabase/ssr";
import type { Database } from "./database.types";

export const createClient = () =>
  createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
```

`src/lib/supabase/server.ts`:
```typescript
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";
import type { Database } from "./database.types";

export const createClient = () => {
  const cookieStore = cookies();
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (cookiesToSet) =>
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          ),
      },
    }
  );
};
```

### B.5 Auth middleware

`src/middleware.ts` protects every route under `(app)/`. Redirect unauthenticated users to `/login`. Refresh session token on every request via `@supabase/ssr` middleware helper.

### B.6 Implement routes

For each route derived from design:
- Server Component by default
- Use `createClient` from `lib/supabase/server.ts` for DB reads
- Suspense boundaries for slow queries
- Server Actions for mutations (annotated `"use server"`)
- Forms: react-hook-form + zod schema derived from entity types
- Optimistic updates for high-frequency actions
- Loading / empty / error states from wireframe annotations

### B.7 PWA setup

`public/manifest.json` derived from approved design system:

```json
{
  "name": "<app name from PROJECT.md>",
  "short_name": "<short name>",
  "theme_color": "<--color-primary>",
  "background_color": "<--color-bg>",
  "display": "standalone",
  "start_url": "/",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

`public/sw.js`:
- Cache-first for static assets
- Network-first for API calls and Supabase data

Register service worker in `app/layout.tsx` via small client component. Add `<link rel="manifest" href="/manifest.json" />` and meta theme-color.

### B.8 Env files

`.env.local` already written by Stage A. Verify it has:
```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
```

Write `.env.example` (commit this):
```
NEXT_PUBLIC_SUPABASE_URL=your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

### B.9 Verification (mandatory)

```bash
cd src && npx tsc --noEmit
cd src && npx eslint . --ext .ts,.tsx --max-warnings 0
```

Both must return zero errors/warnings. Fix everything — don't suppress.

### B.10 Frontend checklist

- [ ] One route per screen from design
- [ ] Every reusable component from design has typed React implementation
- [ ] All forms use react-hook-form + zod with schemas derived from entity types
- [ ] Loading, empty, error states match wireframe annotations
- [ ] Auth middleware protects every `(app)/` route
- [ ] `manifest.json` and `sw.js` configured
- [ ] `tsc --noEmit` zero errors
- [ ] `eslint --max-warnings 0` clean
- [ ] `.env.local` exists and is in `.gitignore`
- [ ] `npm run dev` serves `localhost:3000` without errors

---

## Report and handoff

Update `PROJECT.md` Phase 3 section with:
- Tables created + RLS status
- Routes implemented
- Components built
- Env vars configured
- Handoff notes for QA

Suggest next step: invoke `qa` skill for Phase 4.

## Outputs

| Artifact | Description |
|----------|-------------|
| Supabase project | Tables + RLS + auth configured |
| `supabase/migrations/*.sql` | Versioned migrations |
| `src/` | Complete Next.js + Tailwind + PWA app |
| `src/lib/supabase/database.types.ts` | Generated types |
| `public/manifest.json`, `public/sw.js` | PWA setup |
| `.env.local` | MCP-fetched values (not committed) |
| `.env.example` | Template (committed) |

## Hard rules

- TypeScript strict mode — `"strict": true` in tsconfig.json.
- Never expose Service Role Key on frontend — only Anon Key with RLS.
- Never run untested SQL in production via `execute_sql` — always through migrations.
- If schema needs change later, new migration — never edit applied ones.
- RLS on every table — non-negotiable.
- Never invent screens/entities absent from design.
- Never write manual types for DB rows — always import from `database.types.ts`.
- Run `tsc --noEmit` + `eslint --max-warnings 0` before reporting done.
- App must actually `npm run dev` and serve `localhost:3000` before handoff.

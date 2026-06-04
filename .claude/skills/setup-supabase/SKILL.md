---
name: setup-supabase
description: Sets up the Supabase backend for a PWA — derives schema from the design, applies migrations via Supabase MCP, enables RLS + Auth, and generates TypeScript types. Use when the Engineer needs to configure the database, auth, and policies before scaffolding the Next.js frontend.
---

# Skill: setup-supabase

Configures the full Supabase backend for the PWA project: tables, RLS policies, auth setup, and generated TypeScript types — all via Supabase MCP, no manual credential handling.

## Output language

Chat updates and PROJECT.md notes are written in Brazilian Portuguese (pt-BR). SQL, JSON, identifiers, and migration filenames stay in English.

## Context to load

1. `PROJECT.md` — accumulated decisions, app name
2. `prd.md` — entities listed in the PRD
3. `designs/design.html` — forms and cards reveal the actual fields
4. `wireframes/wireframe.html` — business rules and validation

## Supabase MCP — required tools

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

Never ask the user for Supabase URL, project_ref, or keys — always fetch via MCP.

## Process

### 1. Derive entities

Extract domain entities from the design and PRD:

- Each "noun" the user creates / edits / deletes → one table
- Fields derived from form inputs and card layouts in `designs/design.html`
- Relationships derived from navigation hierarchy (list → item = parent-child)
- Constraints derived from acceptance criteria in `prd.md`

Show the mapping in chat for confirmation before applying anything to the database.

### 2. Inspect current database

```
mcp__supabase__list_tables()
```

If tables already exist, show them and ask whether to drop, modify, or keep them. Never silently overwrite.

### 3. Migration template (per entity)

For each derived entity, build a migration following this template:

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

If `moddatetime` isn't enabled, prepend:
```sql
create extension if not exists moddatetime;
```

### 4. Apply migrations via MCP

For each migration:

```
mcp__supabase__apply_migration(
  name: "00X_<descriptive_name>",
  query: "<SQL content>"
)
```

Also save the SQL to `supabase/migrations/00X_<descriptive_name>.sql` for git versioning.

Confirm with `mcp__supabase__list_tables()` after each apply.

### 5. Auth configuration

Default Supabase Auth setup:
- Email + password sign-up enabled
- Email confirmation enabled
- Anonymous sign-in disabled

If the PRD requires social providers or magic links, enable them via the Supabase dashboard (instruct the user to enable in dashboard since MCP doesn't expose auth config).

### 6. Generate TypeScript types

```
mcp__supabase__generate_typescript_types()
```

Save output to `src/lib/supabase/database.types.ts`. These types are the single source of truth for entity shapes in the frontend — never write manual types for DB rows.

### 7. Run advisors

```
mcp__supabase__get_advisors()
```

Fix every CRITICAL and HIGH finding before reporting done. Common issues: missing RLS, public read policies, missing indexes on foreign keys.

### 8. Output env values

Fetch and save (do NOT print Anon Key to chat in plaintext beyond what's needed):

```
url = mcp__supabase__get_project_url()
keys = mcp__supabase__get_publishable_keys()
```

Write to `.env.local` (must be in `.gitignore`):
```
NEXT_PUBLIC_SUPABASE_URL=<url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<keys.anon_key>
```

## Quality checklist

Before reporting done:

- [ ] Every table has RLS enabled
- [ ] Every table has at least one policy
- [ ] No table is missing `user_id` (unless explicitly shared/public)
- [ ] All foreign keys have indexes
- [ ] `database.types.ts` regenerated and reflects current schema
- [ ] `get_advisors` reports zero CRITICAL/HIGH findings
- [ ] `.env.local` written, `.env.local` is in `.gitignore`

## Outputs

| Artifact | Description |
|----------|-------------|
| Supabase project | Tables + RLS policies + auth configured |
| `supabase/migrations/*.sql` | Versioned migrations for git |
| `src/lib/supabase/database.types.ts` | Generated TypeScript types |
| `.env.local` | Filled with MCP-fetched values (not committed) |

## Notes

- Never expose Service Role Key on the frontend — only Anon Key with RLS active.
- Never run untested SQL in production via `execute_sql` — always go through migrations.
- If schema needs change later, create a new migration — never edit applied migrations.
- RLS on every table is non-negotiable.

---
name: review-code
description: Manual security-focused code review of a PWA codebase. Reads every server action, middleware, query, and form to find input-validation gaps, secret exposure, auth bypass, RLS gaps, XSS, and injection. Use as the first step of Phase 5 (Security) before running the automated audit.
---

# Skill: review-code

Performs a manual, line-level security review of the app produced by Phase 3. Produces a findings list by severity that feeds into `security-audit` for fixes.

## Output language

Findings list, summary, and chat updates in Brazilian Portuguese (pt-BR). File paths, code snippets, function names, and OWASP labels stay in English.

## Context to load

1. `PROJECT.md` — technical decisions, protected routes, env vars, entities
2. `src/middleware.ts` — auth gate
3. `src/lib/supabase/*` — client/server helpers
4. `src/app/**/page.tsx` and `**/actions.ts` — server components, server actions
5. `src/app/**/route.ts` — API routes (if any)
6. `supabase/migrations/*.sql` — schema and RLS policies

## Process

### 1. Inventory

Build a quick map:
- Every server action (`"use server"`)
- Every API route (`src/app/api/**/route.ts`)
- Every middleware-protected route group
- Every direct Supabase query (server-side and client-side)
- Every form (react-hook-form + zod usage)

### 2. Review checklist per category

#### Input validation (every server boundary)

- [ ] Every server action validates input with a zod schema before touching DB
- [ ] Every API route validates request body with zod
- [ ] No `as any` to bypass typing on user input
- [ ] No raw form data passed to Supabase queries

#### Secrets

- [ ] No hardcoded keys, tokens, or URLs in source
- [ ] `NEXT_PUBLIC_*` only used for values that are safe to expose
- [ ] Service Role Key absent from `src/` entirely
- [ ] `.env.local` listed in `.gitignore`
- [ ] No secrets in `console.log` or error messages

#### Auth and access control

- [ ] `src/middleware.ts` runs on every protected route group
- [ ] Every server action calls `createClient()` from `lib/supabase/server.ts` (cookies bound) — never `lib/supabase/client.ts`
- [ ] Every protected page reads user via `supabase.auth.getUser()` before fetching data
- [ ] No server action returns rows that bypass `user_id` filter (RLS catches this, but defense in depth)
- [ ] Logout invalidates session server-side (cookies cleared)

#### RLS policies

- [ ] Every table in `supabase/migrations/*.sql` has `enable row level security`
- [ ] Every table has at least one policy (no table is fully closed by accident)
- [ ] No policy uses `using (true)` without explicit reason
- [ ] `with check` present on insert/update policies
- [ ] Foreign keys to `auth.users` use `not null` where ownership is required

#### XSS

- [ ] No `dangerouslySetInnerHTML` without sanitization
- [ ] No `eval`, `Function(...)`, or `new Function(...)`
- [ ] User-rendered text not interpreted as HTML anywhere

#### SQL injection

- [ ] All queries use Supabase SDK (auto-parameterized) — no raw SQL concatenation with user input
- [ ] If `execute_sql` MCP was used at runtime: never with user input

#### CSRF and headers

- [ ] Server actions rely on Next.js built-in CSRF protection (cookies + origin check) — no custom unprotected POST endpoints
- [ ] Security headers configured in `next.config.js` (will be added by `security-audit` if missing)

#### PWA specific

- [ ] Service Worker does not cache responses from authenticated API calls indiscriminately
- [ ] Web App Manifest does not leak sensitive info
- [ ] Camera/geolocation requested only on demand, not at app load

#### Logging

- [ ] No `console.log` of email, token, password, or PII
- [ ] Error responses don't include stack traces in production

### 3. Findings format

Per finding:

```
FINDING #N
Severity: CRITICAL | HIGH | MEDIUM | LOW
Category: <Input Validation | Secrets | Auth | RLS | XSS | Injection | Headers | PWA | Logging>
File: <path:line>
Description: <what's wrong>
Risk: <what an attacker could do>
Fix: <concrete code-level fix>
```

**Severity guide:**
- **CRITICAL** — direct data exposure, auth bypass, secret leak
- **HIGH** — exploitable with realistic effort, partial bypass
- **MEDIUM** — defense-in-depth gap, hard to exploit alone
- **LOW** — best-practice deviation, no direct exploit

### 4. Summary report

After review, output a summary table in chat:

```markdown
## Review Findings

| Severity | Count |
|----------|-------|
| CRITICAL | N |
| HIGH     | N |
| MEDIUM   | N |
| LOW      | N |

### Top findings
1. [F#1] <description>
2. ...

Hand off to `security-audit` to apply fixes for all CRITICAL/HIGH findings.
```

## Outputs

| Artifact | Description |
|----------|-------------|
| Findings list (chat) | Categorized + prioritized + with fix proposals |

## Notes

- This skill does not apply fixes — `security-audit` does that.
- This skill does not run automated tools — pure manual review.
- If a CRITICAL finding is uncertain, escalate to user before flagging.
- Reading time scales with codebase size — for >100 files, review by category instead of file-by-file.

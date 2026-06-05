---
name: security
description: Phase 5 of the PWA pipeline. Acts as Senior Security Engineer — runs manual code review + OWASP Top 10 audit, fixes vulnerabilities, then deploys to GitHub + Vercel. Use when starting Phase 5, auditing security, reviewing code, or deploying. Triggers on "fase 5", "security", "segurança", "auditoria", "deploy", "vercel", "OWASP".
---

# Skill: security — Security Engineer (Phase 5)

Self-contained Phase 5. Three stages: (A) manual code review, (B) OWASP audit + fixes, (C) deploy to GitHub + Vercel.

## Output language

Findings, audit report, status updates, PROJECT.md notes in pt-BR. File paths, code snippets, OWASP labels, CVE IDs, CLI output stay verbatim in English.

## Context to load

1. `PROJECT.md` — technical decisions, protected routes, env vars, entities
2. `src/middleware.ts` — auth gate
3. `src/lib/supabase/*` — client/server helpers
4. `src/app/**/page.tsx` and `**/actions.ts` — server components + actions
5. `src/app/**/route.ts` — API routes (if any)
6. `supabase/migrations/*.sql` — schema and RLS policies

Phase 4 must be done with zero CRITICAL/HIGH bugs open. If not, instruct user to run `qa` skill first.

---

## STAGE A — Manual code review

### A.1 Inventory

Build a quick map:
- Every server action (`"use server"`)
- Every API route (`src/app/api/**/route.ts`)
- Every middleware-protected route group
- Every direct Supabase query (server-side and client-side)
- Every form (react-hook-form + zod usage)

### A.2 Review checklist per category

#### Input validation (every server boundary)
- [ ] Every server action validates input with zod schema before touching DB
- [ ] Every API route validates request body with zod
- [ ] No `as any` to bypass typing on user input
- [ ] No raw form data passed to Supabase queries

#### Secrets
- [ ] No hardcoded keys, tokens, URLs in source
- [ ] `NEXT_PUBLIC_*` only for values safe to expose
- [ ] Service Role Key absent from `src/` entirely
- [ ] `.env.local` in `.gitignore`
- [ ] No secrets in `console.log` or error messages

#### Auth and access control
- [ ] `src/middleware.ts` runs on every protected route group
- [ ] Every server action calls `createClient()` from `lib/supabase/server.ts` — never `lib/supabase/client.ts`
- [ ] Every protected page reads user via `supabase.auth.getUser()` before fetching data
- [ ] No server action returns rows bypassing `user_id` filter (defense in depth)
- [ ] Logout invalidates session server-side (cookies cleared)

#### RLS policies
- [ ] Every table in `supabase/migrations/*.sql` has `enable row level security`
- [ ] Every table has at least one policy
- [ ] No policy uses `using (true)` without explicit reason
- [ ] `with check` present on insert/update policies
- [ ] Foreign keys to `auth.users` use `not null` where ownership required

#### XSS
- [ ] No `dangerouslySetInnerHTML` without sanitization
- [ ] No `eval`, `Function(...)`, `new Function(...)`
- [ ] User-rendered text not interpreted as HTML anywhere

#### SQL injection
- [ ] All queries use Supabase SDK (auto-parameterized) — no raw SQL concatenation with user input
- [ ] If `execute_sql` MCP used at runtime: never with user input

#### CSRF and headers
- [ ] Server actions rely on Next.js built-in CSRF (cookies + origin check) — no custom unprotected POST endpoints
- [ ] Security headers configured in `next.config.js` (Stage B applies if missing)

#### PWA specific
- [ ] Service Worker doesn't cache authenticated API calls indiscriminately
- [ ] Web App Manifest doesn't leak sensitive info
- [ ] Camera/geolocation requested only on demand

#### Logging
- [ ] No `console.log` of email, token, password, or PII
- [ ] Error responses don't include stack traces in production

### A.3 Findings format

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

### A.4 Summary

Output table in chat. Carry findings forward to Stage B.

---

## STAGE B — OWASP audit + fixes

### B.1 OWASP Top 10 checklist

#### A01 — Broken Access Control
- [ ] Every Supabase table has RLS enabled (`mcp__supabase__get_advisors` confirms)
- [ ] Every route in `(app)/` checks session before render
- [ ] No Server Action / API Route exposes other-user data
- [ ] `createServerClient` correctly used in RSC

#### A02 — Cryptographic Failures
- [ ] No hardcoded secrets in source
- [ ] `NEXT_PUBLIC_` only for truly public data
- [ ] Supabase Anon Key only used where RLS protects
- [ ] `.env.local` and `.env*.local` in `.gitignore`

#### A03 — Injection
- [ ] All queries use Supabase SDK (auto-parameterized)
- [ ] No SQL string concatenation with user input
- [ ] DOM inputs sanitized before render (XSS)
- [ ] `dangerouslySetInnerHTML` absent or sanitized

#### A05 — Security Misconfiguration
- [ ] Security headers configured in `next.config.js`
- [ ] CSP defined (or explicitly deferred with justification)
- [ ] `X-Frame-Options: SAMEORIGIN`
- [ ] `Permissions-Policy` restricts unused APIs

#### A06 — Vulnerable Components
- [ ] `npm audit --audit-level=high` clean
- [ ] Deps updated to versions without known CVEs

#### A07 — Authentication Failures
- [ ] Supabase Auth configured with email confirmation
- [ ] Logout clears session client + invalidates server-side
- [ ] Middleware protects every `(app)/` route

#### A09 — Logging Failures
- [ ] No `console.log` with sensitive data
- [ ] Error logs don't expose stack traces to end user

#### PWA-specific
- [ ] Service Worker doesn't cache authenticated API responses incorrectly
- [ ] Camera (BarcodeDetector) requests permission on demand
- [ ] Web Share API doesn't leak sensitive data

### B.2 Automated checks (in sequence — stop and fix on failure)

```bash
cd src && npx tsc --noEmit
cd src && npx eslint . --ext .ts,.tsx --max-warnings 0
cd src && npm audit --audit-level=high

# Secret scan in source
git grep -nE "service_role|SUPABASE_SERVICE|password\s*=\s*['\"]|api_key\s*=\s*['\"]" -- '*.ts' '*.tsx' '*.js' '*.json' || echo "Clean"

# Secret scan in git history
git log --all -p | grep -nE "service_role|SUPABASE_SERVICE" || echo "Clean"
```

### B.3 Supabase advisors

```
mcp__supabase__get_advisors()
```

Fix every CRITICAL and HIGH finding before continuing.

### B.4 Apply security headers

If `src/next.config.js` lacks them, add:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-DNS-Prefetch-Control', value: 'on' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Permissions-Policy', value: 'camera=(self), microphone=(), geolocation=()' },
        ],
      },
    ];
  },
};
module.exports = nextConfig;
```

Adjust `Permissions-Policy` based on PRD: only allow features actually used.

### B.5 Apply Stage A fixes

For every CRITICAL/HIGH finding from Stage A:
- Apply suggested fix
- Re-run `tsc --noEmit` after each batch
- Move MEDIUM/LOW to "known issues" if not addressable

### B.6 Pre-deploy gate (hard checklist)

- [ ] `tsc --noEmit` zero errors
- [ ] `eslint --max-warnings 0` clean
- [ ] `npm audit --audit-level=high` clean
- [ ] No secrets in source or git history
- [ ] RLS active on every table
- [ ] `.env.local` in `.gitignore`
- [ ] `middleware.ts` protects `(app)/`
- [ ] Security headers configured
- [ ] Zero CRITICAL/HIGH findings from Stage A remain open

**If any item fails: fix before continuing. No deploy with vulnerabilities open.**

### B.7 Audit report

```markdown
## Resultado da Auditoria

- OWASP A01 (Access Control): ✅
- OWASP A02 (Cryptography): ✅
- OWASP A03 (Injection): ✅
- OWASP A05 (Misconfiguration): ✅
- OWASP A06 (Vulnerabilities): ✅
- OWASP A07 (Auth): ✅
- OWASP A09 (Logging): ✅
- npm audit: ✅ (zero high/critical)
- Supabase advisors: ✅
- tsc --noEmit: ✅
- eslint: ✅

### Findings Fixed
- [F#1] <desc> — CRITICAL — fixed
- ...

### Known Issues (low priority)
- ...
```

---

## STAGE C — Deploy to GitHub + Vercel

### C.1 Preconditions (hard gates)

- [ ] Stage B passed clean — zero CRITICAL/HIGH open
- [ ] `tsc --noEmit` zero errors
- [ ] `npm audit --audit-level=high` clean
- [ ] `.env.local` exists with `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- [ ] `.env.local` in `.gitignore`
- [ ] `gh auth status` shows authenticated user
- [ ] `vercel --version` works (CLI installed)

If any gate fails: stop and instruct user.

### C.2 Derive names

From `PROJECT.md`:
- Repo name: app name in kebab-case (e.g. "Estoque Doméstico" → `estoque-domestico-pwa`)
- Vercel project name: same

### C.3 Pre-flight verification

```bash
grep -q "^\.env\.local$\|^\.env\*\.local$" .gitignore && echo "OK" || echo "MISSING — fix .gitignore first"
git -C src diff --cached | grep -E "SUPABASE_SERVICE|service_role" && echo "ABORT" || echo "Clean"
```

### C.4 Create GitHub repo and push

```bash
gh repo create <repo-name> \
  --private \
  --source=src \
  --remote=origin \
  --push \
  --description "PWA built with Next.js + Supabase"
```

Verify `.env.local` not pushed:
```bash
git -C src ls-files | grep "^\.env" | grep -v "\.example$" && echo "LEAK" || echo "Clean"
```

If "LEAK": stop, instruct user how to scrub.

### C.5 Deploy to Vercel

```bash
cd src && vercel --prod --yes
```

Capture production URL.

### C.6 Configure env vars on Vercel

```bash
url=$(grep NEXT_PUBLIC_SUPABASE_URL .env.local | cut -d'=' -f2)
key=$(grep NEXT_PUBLIC_SUPABASE_ANON_KEY .env.local | cut -d'=' -f2)

cd src && echo "$url" | vercel env add NEXT_PUBLIC_SUPABASE_URL production
cd src && echo "$key" | vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production
```

If non-interactive piping fails, fall back to interactive:
```bash
cd src && vercel env add NEXT_PUBLIC_SUPABASE_URL production
cd src && vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production
```

### C.7 Re-deploy with env vars active

```bash
cd src && vercel --prod --yes
```

Capture final production URL.

### C.8 Configure Supabase redirect URLs

Manual step — instruct user. Add Vercel production URL to Supabase dashboard:
- Authentication → URL Configuration → Site URL
- Authentication → URL Configuration → Redirect URLs

Without this, auth callbacks fail in production.

### C.9 Post-deploy smoke test

Open production URL via Chrome DevTools MCP:
- [ ] App loads without console errors
- [ ] Auth (signup or login) works end-to-end
- [ ] One CRUD round-trip per main entity works
- [ ] PWA installable (manifest + SW detected in DevTools → Application)
- [ ] HTTPS active
- [ ] Security headers present:
  ```bash
  curl -sI https://<app>.vercel.app | grep -E "X-Frame-Options|X-Content-Type-Options|Referrer-Policy"
  ```

If any fails: report and roll back if necessary.

### C.10 Final report

```markdown
## Deploy concluído

- **GitHub:** https://github.com/<user>/<repo-name> (privado)
- **Vercel:** https://<app>.vercel.app
- **Env vars configuradas:** NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
- **Smoke test:** ✅ todos os checks passaram

### Ação manual restante
Adicionar Vercel URL como Site URL + Redirect URL no Supabase Auth (dashboard).
```

Update `PROJECT.md` Phase 5 section with: GitHub repo URL, Vercel URL, audit summary.

## Outputs

| Artifact | Description |
|----------|-------------|
| Findings + Audit report (chat) | OWASP checklist + fixed findings |
| Fixed code | All CRITICAL/HIGH findings patched |
| `next.config.js` | Security headers |
| GitHub repo | Private, source code pushed |
| Vercel project | Production deploy with env vars |
| Production URL | HTTPS app live |

## Hard rules

- Never deploy with `npm audit --audit-level=high` failing.
- Never deploy with `tsc --noEmit` errors.
- RLS on every table — non-negotiable.
- Repos private by default (global CLAUDE.md rule).
- Auth via `gh auth setup-git` — never Git Credential Manager.
- Never push `.env.local` — verify before every push.
- If user has separate Vercel account, instruct `vercel switch` before starting.

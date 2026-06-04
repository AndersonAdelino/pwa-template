---
name: security-audit
description: Runs the OWASP Top 10 audit on the PWA — verifies access control, secrets, injection, misconfiguration, vulnerable deps, auth, logging. Also applies security headers and fixes findings from review-code. Use as the second step of Phase 5 (after review-code, before deploy-vercel).
---

# Skill: security-audit

Runs the automated portion of Phase 5: OWASP Top 10 checklist, `npm audit`, secret scan in code + git history, security headers verification, RLS verification via Supabase advisors. Applies fixes for every CRITICAL/HIGH issue before reporting done.

## Output language

Audit report, status updates, and PROJECT.md notes in Brazilian Portuguese (pt-BR). OWASP labels, CVE IDs, file paths, and CLI output quoted verbatim in English.

## Context to load

1. `PROJECT.md` — Phase 3 decisions, env vars, routes
2. Findings list from `review-code` (in chat history)
3. `src/next.config.js` — current header config
4. `src/package.json` — deps
5. `supabase/migrations/*.sql` — schema

## OWASP Top 10 checklist

### A01 — Broken Access Control
- [ ] Every Supabase table has RLS enabled (`mcp__supabase__get_advisors` confirms)
- [ ] Every route in `(app)/` checks session before render
- [ ] No Server Action or API Route exposes other-user data
- [ ] `createServerClient` correctly used in RSC

### A02 — Cryptographic Failures
- [ ] No hardcoded secrets in source
- [ ] `NEXT_PUBLIC_` only for truly public data
- [ ] Supabase Anon Key only used where RLS protects
- [ ] `.env.local` and `.env*.local` in `.gitignore`

### A03 — Injection
- [ ] All queries use Supabase SDK (auto-parameterized)
- [ ] No SQL string concatenation with user input
- [ ] DOM inputs sanitized before render (XSS)
- [ ] `dangerouslySetInnerHTML` absent or sanitized

### A05 — Security Misconfiguration
- [ ] Security headers configured in `next.config.js`
- [ ] CSP defined (or explicitly deferred with justification)
- [ ] `X-Frame-Options: SAMEORIGIN`
- [ ] `Permissions-Policy` restricts unused APIs

### A06 — Vulnerable Components
- [ ] `npm audit --audit-level=high` clean
- [ ] Deps updated to versions without known CVEs

### A07 — Authentication Failures
- [ ] Supabase Auth configured with email confirmation
- [ ] Logout clears session client + invalidates server-side
- [ ] Middleware protects every `(app)/` route

### A09 — Logging Failures
- [ ] No `console.log` with sensitive data (email, token, password)
- [ ] Error logs don't expose stack traces to end user

### PWA-specific
- [ ] Service Worker doesn't cache authenticated API responses incorrectly
- [ ] Camera (BarcodeDetector) requests permission on demand
- [ ] Web Share API doesn't leak sensitive data

## Process

### 1. Automated checks

Run in sequence (stop and fix on failure):

```bash
# Zero TypeScript errors
cd src && npx tsc --noEmit

# Zero ESLint warnings
cd src && npx eslint . --ext .ts,.tsx --max-warnings 0

# Dependency audit — no high/critical
cd src && npm audit --audit-level=high

# Secret scan in source
git grep -nE "service_role|SUPABASE_SERVICE|password\s*=\s*['\"]|api_key\s*=\s*['\"]" -- '*.ts' '*.tsx' '*.js' '*.json' || echo "Clean"

# Secret scan in git history
git log --all -p | grep -nE "service_role|SUPABASE_SERVICE" || echo "Clean"
```

### 2. Supabase advisors

```
mcp__supabase__get_advisors()
```

Fix every CRITICAL and HIGH advisory finding before continuing.

### 3. Apply security headers

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

### 4. Apply review-code fixes

For every CRITICAL/HIGH finding from `review-code`:
- Apply the suggested fix
- Re-run `tsc --noEmit` after each batch
- Move MEDIUM/LOW to "known issues" if not addressable in this pass

### 5. Pre-deploy gate

Confirm full checklist:

- [ ] `tsc --noEmit` zero errors
- [ ] `eslint --max-warnings 0` clean
- [ ] `npm audit --audit-level=high` clean
- [ ] No secrets in source or git history
- [ ] RLS active on every table (`get_advisors` clean)
- [ ] `.env.local` in `.gitignore`
- [ ] `middleware.ts` protects `(app)/`
- [ ] Security headers configured
- [ ] Zero CRITICAL/HIGH findings from `review-code` remain open

**If any item fails: fix before continuing. No deploy with vulnerabilities open.**

### 6. Audit report

Output in chat:

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

## Outputs

| Artifact | Description |
|----------|-------------|
| Audit report (chat) | OWASP checklist results + fixed findings |
| Fixed code | All CRITICAL/HIGH findings patched in-place |
| `next.config.js` | Security headers configured |

## Notes

- Never deploy with `npm audit --audit-level=high` failing.
- Never deploy with `tsc --noEmit` errors.
- RLS on every table is non-negotiable — no exceptions.
- Hand off to `deploy-vercel` only after this passes clean.

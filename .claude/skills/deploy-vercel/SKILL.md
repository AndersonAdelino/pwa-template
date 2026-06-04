---
name: deploy-vercel
description: Deploys the PWA to production — creates a private GitHub repo via gh CLI, pushes the code, then deploys to Vercel with env vars wired from the local .env.local. Use as the final step of Phase 5, only after security-audit passes clean.
---

# Skill: deploy-vercel

Ships the PWA to production. Creates a private GitHub repo via `gh` CLI, pushes `src/`, deploys to Vercel, configures env vars from `.env.local`, and runs a post-deploy smoke test.

## Output language

Status updates, deploy URLs, and PROJECT.md notes in Brazilian Portuguese (pt-BR). CLI commands, repo names, and URLs stay verbatim.

## Context to load

1. `PROJECT.md` — app name (for repo + Vercel project name)
2. `.env.local` — env values to copy into Vercel
3. `.gitignore` — confirm `.env.local` is listed

## Preconditions (hard gates)

Before running anything:
- [ ] `security-audit` passed clean — zero CRITICAL/HIGH findings open
- [ ] `tsc --noEmit` zero errors
- [ ] `npm audit --audit-level=high` clean
- [ ] `.env.local` exists with `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- [ ] `.env.local` listed in `.gitignore`
- [ ] `gh auth status` shows authenticated user
- [ ] `vercel --version` works (CLI installed)

If any gate fails: stop and instruct the user to fix it.

## Process

### 1. Derive names

From `PROJECT.md`:
- Repo name: app name in kebab-case (e.g. "Estoque Doméstico" → `estoque-domestico-pwa`)
- Vercel project name: same as repo name

### 2. Pre-flight verification

```bash
# Confirm .env.local is ignored
grep -q "^\.env\.local$\|^\.env\*\.local$" .gitignore && echo "OK" || echo "MISSING — fix .gitignore first"

# Confirm no secrets staged
git -C src diff --cached | grep -E "SUPABASE_SERVICE|service_role" && echo "ABORT" || echo "Clean"
```

### 3. Create GitHub repo and push

```bash
gh repo create <repo-name> \
  --private \
  --source=src \
  --remote=origin \
  --push \
  --description "PWA built with Next.js + Supabase"
```

Verify `.env.local` was not pushed:
```bash
git -C src ls-files | grep "^\.env" | grep -v "\.example$" && echo "LEAK" || echo "Clean"
```

If "LEAK": stop, instruct user how to scrub.

### 4. Deploy to Vercel

```bash
# First deploy (will prompt for project setup)
cd src && vercel --prod --yes
```

Capture the production URL from the CLI output.

### 5. Configure env vars on Vercel

Read values from local `.env.local` and push to Vercel production:

```bash
# Read from .env.local
url=$(grep NEXT_PUBLIC_SUPABASE_URL .env.local | cut -d'=' -f2)
key=$(grep NEXT_PUBLIC_SUPABASE_ANON_KEY .env.local | cut -d'=' -f2)

# Push to Vercel (non-interactive)
cd src && echo "$url" | vercel env add NEXT_PUBLIC_SUPABASE_URL production
cd src && echo "$key" | vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production
```

If non-interactive piping fails, fall back to:
```bash
cd src && vercel env add NEXT_PUBLIC_SUPABASE_URL production
cd src && vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production
```
And paste values interactively.

### 6. Re-deploy with env vars active

```bash
cd src && vercel --prod --yes
```

Capture final production URL.

### 7. Configure Supabase redirect URLs

Open Supabase dashboard (manual step — instruct the user) and add the Vercel production URL to:
- Authentication → URL Configuration → Site URL
- Authentication → URL Configuration → Redirect URLs

Without this, auth callbacks will fail in production.

### 8. Post-deploy smoke test

Open production URL in a browser (via Chrome DevTools MCP):

- [ ] App loads without console errors
- [ ] Auth (signup or login) works end-to-end
- [ ] One CRUD round-trip per main entity works
- [ ] PWA installable (manifest + SW detected in DevTools → Application)
- [ ] HTTPS active (Vercel default)
- [ ] Security headers present:
  ```bash
  curl -sI https://<app>.vercel.app | grep -E "X-Frame-Options|X-Content-Type-Options|Referrer-Policy"
  ```

If any check fails: report to user and roll back if necessary.

### 9. Report

Output in chat:

```markdown
## Deploy concluído

- **GitHub:** https://github.com/<user>/<repo-name> (privado)
- **Vercel:** https://<app>.vercel.app
- **Env vars configuradas:** NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
- **Smoke test:** ✅ todos os checks passaram

### Ação manual restante
Adicionar Vercel URL como Site URL + Redirect URL no Supabase Auth (dashboard).
```

## Outputs

| Artifact | Description |
|----------|-------------|
| GitHub repo | Private, source code pushed |
| Vercel project | Production deploy with env vars |
| Production URL | HTTPS app live |

## Notes

- Repos private by default (global CLAUDE.md rule).
- Auth via `gh auth setup-git` — never Git Credential Manager.
- Never push `.env.local` — verify before every push.
- Vercel CLI must be installed and authenticated before this skill runs.
- If user has a separate Vercel account, instruct them to `vercel switch` before starting.

---
description: Phase 3 — invokes the engineer subagent to build the Next.js + Supabase + PWA app.
argument-hint: [foco ou feature específica — opcional]
---

Invoke the `engineer` subagent via the Agent tool. Pass `$ARGUMENTS` as scope hint if provided.

The subagent owns Phase 3: derive entities/routes from design, run `setup-supabase` skill, run `implement-frontend` skill, verify with `tsc --noEmit` and `eslint`, update `PROJECT.md`.

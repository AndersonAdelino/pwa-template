---
description: Phase 5 — invokes the security subagent to audit the code, fix vulnerabilities, and deploy to GitHub + Vercel.
---

Invoke the `security` subagent via the Agent tool.

The subagent owns Phase 5: run `review-code` skill, run `security-audit` skill, run `deploy-vercel` skill, smoke-test production, update `PROJECT.md`.

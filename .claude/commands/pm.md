---
description: Phase 1 — invokes the product-manager subagent to write the PRD and Lo-Fi wireframe.
argument-hint: [descrição curta do app — opcional]
---

Invoke the `product-manager` subagent via the Agent tool. Pass `$ARGUMENTS` as the initial brief (app description) if provided; otherwise let the agent ask the user.

The subagent owns Phase 1 of the PWA pipeline: write `prd.md`, then generate `wireframes/wireframe.html`, then update `PROJECT.md`.

---
description: Phase 4 — invokes the qa subagent to test all flows in a real browser and fix bugs.
argument-hint: [fluxo ou tela específica para focar — opcional]
---

Invoke the `qa` subagent via the Agent tool. Pass `$ARGUMENTS` as scope hint if provided.

The subagent owns Phase 4: run the 6-flow suite via Chrome DevTools MCP, log bugs by severity, fix CRITICAL/HIGH in-place, iterate until clean, update `PROJECT.md`.

---
name: qa
description: Phase 4 of the PWA pipeline. Acts as Senior QA Engineer — opens the app in a real browser via Chrome DevTools MCP, exercises every flow (auth, CRUD, navigation, responsive, PWA, network), logs bugs with severity, fixes them, iterates until the golden path is clean. Use when starting Phase 4, testing the app, validating flows. Triggers on "fase 4", "qa", "testar", "validar".
---

# Skill: qa — QA Engineer (Phase 4)

Self-contained Phase 4. Drives the app in a real browser via Chrome DevTools MCP. Runs 6 mandatory flows, logs every bug, fixes CRITICAL/HIGH in-place, re-runs affected flows, exits only when zero CRITICAL/HIGH remain.

## Output language

Bug reports, status updates, PROJECT.md notes in pt-BR. Console errors and stack traces quoted verbatim in original language.

## Context to load

1. `PROJECT.md` — app URL (dev or prod), implemented routes, domain entities, test credentials
2. `prd.md` — acceptance criteria → test cases
3. `wireframes/wireframe.html` — expected flows, dynamic states, business rules
4. `designs/design.html` — loading/empty/error states

Phase 3 must be done. If not, instruct user to run `engineer` skill first.

## Chrome DevTools MCP — required tools

| Need | MCP tool |
|------|----------|
| Open new tab | `mcp__chrome-devtools__new_page` |
| Navigate to URL | `mcp__chrome-devtools__navigate_page` |
| Click element | `mcp__chrome-devtools__click` |
| Fill field | `mcp__chrome-devtools__fill` |
| Fill form | `mcp__chrome-devtools__fill_form` |
| Submit / key press | `mcp__chrome-devtools__press_key` |
| Screenshot | `mcp__chrome-devtools__take_screenshot` |
| Read console | `mcp__chrome-devtools__list_console_messages` |
| Read network | `mcp__chrome-devtools__list_network_requests` |
| Wait for element/network | `mcp__chrome-devtools__wait_for` |
| Emulate mobile | `mcp__chrome-devtools__emulate` |
| Run JS | `mcp__chrome-devtools__evaluate_script` |
| Resize window | `mcp__chrome-devtools__resize_page` |

## Process

### 1. Preparation

- Extract from PROJECT.md: app URL, route list, entity list, test credentials (create new account if none)
- If app URL is local (`http://localhost:3000`), confirm dev server is running. If not, start it: `cd src && npm run dev`
- Open browser and navigate to URL
- If first screen is blank or 500: stop. Report failure, don't run further tests.

### 2. Flow suite (6 flows)

For every test: screenshot before + after + record console errors and network failures.

#### FLOW 1 — Authentication

**Signup:**
- [ ] Fill email + password → submit
- [ ] Success feedback or correct redirect
- [ ] Zero console errors

**Login:**
- [ ] Valid credentials → enters app
- [ ] Invalid credentials → visible error message
- [ ] Empty fields → client-side validation before submit

**Logout:**
- [ ] Logout button visible and functional
- [ ] Redirects to login after logout
- [ ] Protected route after logout → redirect to login (not 404 or blank)

#### FLOW 2 — CRUD per domain entity

For every entity from PROJECT.md:

**Create:**
- [ ] Open creation form
- [ ] Fill all required fields → submit → item appears in list
- [ ] Submit with empty required fields → validation shown
- [ ] Special characters: `<script>alert(1)</script>`, `"'`, emojis 🧪 → does not break

**Read / List:**
- [ ] List loads without error
- [ ] Empty state shown when no items
- [ ] Loading state shown while data fetches

**Update:**
- [ ] Open edit modal/screen
- [ ] Change a field → save → change reflected in list
- [ ] Cancel edit → no change persisted

**Delete:**
- [ ] Confirmation before delete
- [ ] Item removed after confirmation
- [ ] Cancel delete → item remains

#### FLOW 3 — Navigation and routing

- [ ] Every PROJECT.md route responds (no 404)
- [ ] Inter-screen navigation without full page reload (SPA behavior)
- [ ] Browser back button works
- [ ] Direct URL to protected route without auth → redirect to login

#### FLOW 4 — Mobile responsiveness

```
mcp__chrome-devtools__emulate({ device: "iPhone 12" })
```

- [ ] Layout doesn't break on small screens
- [ ] Tap targets ≥ 44×44px
- [ ] Forms usable on mobile
- [ ] Scroll works correctly
- [ ] No clipped or overlapping elements

Return to desktop after.

#### FLOW 5 — PWA

- [ ] Manifest loaded: `evaluate_script("JSON.stringify(await fetch('/manifest.json').then(r=>r.json()))")`
- [ ] Service Worker registered: `evaluate_script("navigator.serviceWorker.getRegistrations().then(r=>r.map(s=>s.scope))")`
- [ ] Manifest icons present in `/public/`
- [ ] `display: standalone` in manifest

#### FLOW 6 — Network and security smoke

After full CRUD:
- [ ] No 4xx or 5xx requests (except expected 401 on logout)
- [ ] No request exposes `service_role` key
- [ ] Auth headers present on protected requests
- [ ] Other-user data does not return (RLS working)

### 3. Bug logging

Per bug:

```
BUG #N
Severity: CRITICAL | HIGH | MEDIUM | LOW
Flow: <flow name>
Reproduction steps:
  1. ...
  2. ...
Actual result: <what happens>
Expected result: <what should happen>
Console error: <exact message if any>
Screenshot: <description>
```

**Severities:**
- **CRITICAL** — app breaks, data lost, user cannot complete main flow
- **HIGH** — important feature broken but workaround exists
- **MEDIUM** — wrong behavior but doesn't block usage
- **LOW** — UI/UX polish, wrong text, minor layout

### 4. Fix cycle

If CRITICAL or HIGH bugs found:

1. Report bug list to user with severities
2. Fix bugs directly in code
3. Re-run only affected flows
4. Repeat until zero CRITICAL and HIGH bugs
5. MEDIUM and LOW: fix if quick, otherwise log under "known issues" in PROJECT.md

**Never advance to Phase 5 with CRITICAL or HIGH bugs open.**

### 5. Post-fix verification

If any file modified during fixes:

```bash
cd src && npx tsc --noEmit
cd src && npx eslint . --ext .ts,.tsx --max-warnings 0
```

Zero errors mandatory.

### 6. Report and handoff

Update `PROJECT.md` Phase 4 section with:
- Flows tested with ✅/❌ each
- Bugs found by severity (count)
- Bugs fixed in-session
- Known issues remaining (MEDIUM/LOW)

Suggest next step: invoke `security` skill for Phase 5.

## Outputs

| Artifact | Description |
|----------|-------------|
| App validated | All 6 flows pass without CRITICAL/HIGH bugs |
| Fixed code | CRITICAL/HIGH bugs patched in-place |

## Hard rules

- Always screenshot on bug — visual evidence mandatory.
- Test with real data on real Supabase from Phase 3 — never mocks.
- Special-character XSS edge cases mandatory.
- Mobile not optional — PWA must work on small screens.
- If app fails to start: stop and report.
- Pixel-perfect bugs out of scope — focus on functionality.

---
name: run-qa-tests
description: Runs the full QA suite on the PWA app via Chrome DevTools MCP — opens the app in a real browser, exercises auth + CRUD + navigation + responsive + PWA + network flows, logs bugs with severity, applies fixes, and iterates until the golden path is clean. Use when a QA Engineer needs to validate the implemented app before security review.
---

# Skill: run-qa-tests

Drives the app in a real browser via Chrome DevTools MCP. Runs 6 mandatory flows, logs every bug with severity and reproduction steps, applies fixes for CRITICAL/HIGH bugs in-place, re-runs affected flows, and only exits when zero CRITICAL/HIGH bugs remain.

## Output language

Bug reports, status updates, and PROJECT.md notes in Brazilian Portuguese (pt-BR). Console errors and stack traces quoted verbatim in their original language.

## Context to load

1. `PROJECT.md` — app URL (dev or prod), implemented routes, domain entities, test credentials if any
2. `prd.md` — acceptance criteria → become test cases
3. `wireframes/wireframe.html` — expected flows, dynamic states, business rules
4. `designs/design.html` — loading/empty/error states

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
| Read console errors | `mcp__chrome-devtools__list_console_messages` |
| Read network requests | `mcp__chrome-devtools__list_network_requests` |
| Wait for element/network | `mcp__chrome-devtools__wait_for` |
| Emulate mobile | `mcp__chrome-devtools__emulate` |
| Run JS | `mcp__chrome-devtools__evaluate_script` |
| Resize window | `mcp__chrome-devtools__resize_page` |

## Process

### 1. Preparation

- Extract from PROJECT.md: app URL, route list, entity list, test credentials (create new account if none)
- If app URL is local (`http://localhost:3000`), confirm dev server is running. If not, start it: `cd src && npm run dev`
- Open browser and navigate to URL
- If first screen is blank or 500: stop. Report failure and don't run further tests.

### 2. Flow suite (6 flows)

For every test: screenshot before + after + record console errors and network failures.

---

#### FLOW 1 — Authentication

**Signup:**
- [ ] Fill email + password → submit
- [ ] Success feedback or correct redirect
- [ ] Zero console errors

**Login:**
- [ ] Valid credentials → enters app
- [ ] Invalid credentials → visible error message (not just console)
- [ ] Empty fields → client-side validation before submit

**Logout:**
- [ ] Logout button visible and functional
- [ ] Redirects to login after logout
- [ ] Protected route after logout → redirect to login (not 404 or blank)

---

#### FLOW 2 — CRUD per domain entity

For every entity from PROJECT.md:

**Create:**
- [ ] Open creation form
- [ ] Fill all required fields → submit → item appears in list
- [ ] Submit with empty required fields → validation shown
- [ ] Special characters: `<script>alert(1)</script>`, `"'`, emojis 🧪 → does not break

**Read / List:**
- [ ] List loads without error
- [ ] Empty state shown when no items (not blank screen)
- [ ] Loading state shown while data fetches

**Update:**
- [ ] Open edit modal/screen
- [ ] Change a field → save → change reflected in list
- [ ] Cancel edit → no change persisted

**Delete:**
- [ ] Confirmation before delete (no immediate delete)
- [ ] Item removed from list after confirmation
- [ ] Cancel delete → item remains

---

#### FLOW 3 — Navigation and routing

- [ ] Every PROJECT.md route responds (no 404)
- [ ] Inter-screen navigation without full page reload (SPA behavior)
- [ ] Browser back button works correctly
- [ ] Direct URL to protected route without auth → redirect to login

---

#### FLOW 4 — Mobile responsiveness

Emulate iPhone 12 (or similar):
```
mcp__chrome-devtools__emulate({ device: "iPhone 12" })
```

- [ ] Layout doesn't break on small screens
- [ ] Tap targets ≥ 44×44px
- [ ] Forms usable on mobile
- [ ] Scroll works correctly
- [ ] No clipped or overlapping elements

Return to desktop after.

---

#### FLOW 5 — PWA

- [ ] Manifest loaded: `evaluate_script("JSON.stringify(await fetch('/manifest.json').then(r=>r.json()))")`
- [ ] Service Worker registered: `evaluate_script("navigator.serviceWorker.getRegistrations().then(r=>r.map(s=>s.scope))")`
- [ ] Manifest icons present in `/public/`
- [ ] `display: standalone` in manifest

---

#### FLOW 6 — Network and security smoke

After running full CRUD:
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
2. Fix bugs directly in code (do not exit skill)
3. Re-run only the affected flows
4. Repeat until zero CRITICAL and HIGH bugs
5. MEDIUM and LOW bugs: fix if quick, otherwise log under "known issues" in PROJECT.md

**Never advance to security with CRITICAL or HIGH bugs open.**

### 5. Post-fix verification

If any file was modified during fixes:

```bash
cd src && npx tsc --noEmit
cd src && npx eslint . --ext .ts,.tsx --max-warnings 0
```

Zero errors mandatory.

### 6. Report

Summarize in chat (and PROJECT.md update will be written by the QA agent, not this skill):

- Flows tested with ✅/❌ each
- Bugs found by severity (count)
- Bugs fixed in-session
- Known issues remaining (MEDIUM/LOW)

## Outputs

| Artifact | Description |
|----------|-------------|
| App validated | All 6 flows pass without CRITICAL/HIGH bugs |
| Fixed code | Any CRITICAL/HIGH bugs patched in-place |

## Notes

- Always screenshot on bug — visual evidence mandatory.
- Test with real data on real Supabase from Phase 3 — never mocks.
- Special-character XSS edge cases mandatory.
- Mobile not optional — PWA must work on small screens.
- If app fails to start: stop and report — don't run further tests.
- Pixel-perfect bugs are out of scope here — focus on functionality.

---
name: generate-wireframes
description: Generates a navigable Lo-Fi wireframe HTML file from an approved PRD. Use when a Product Manager has the PRD ready and needs to produce the visual structure (screens, flow, dynamic states, annotations for 5 audiences) before handing off to the Designer.
---

# Skill: generate-wireframes

Produces `wireframes/wireframe.html` — a single self-contained HTML file with a navigable Lo-Fi wireframe and PM annotations for 5 audiences.

## Output language

All wireframe annotations, screen labels, and chat updates are in Brazilian Portuguese (pt-BR). CSS / HTML / JS identifiers stay in English.

## Context to load

1. `prd.md` — entities, user stories, screens listed
2. `PROJECT.md` — any prior decisions
3. `wireframes/wireframe.html` if it exists — iteration mode
4. **Mandatory:** `.claude/skills/generate-wireframes/wireframes.md` — full PM methodology reference

Read `wireframes.md` before producing anything — it defines fidelity strategy, annotation patterns, accessibility rules, and the 10-item PM checklist.

## Process

### 1. Apply methodology (wireframes.md §1–§3)

Before any code:

- **Fidelity:** Discovery / validation phase → Lo-Fi mandatory.
- **Storyboard:** Map the full screen flow, not isolated screens.
- **Real content:** Never Lorem Ipsum — use real data from the app's domain.
- **Grid:** 8-point system (8/16/24/32px spacing), mobile-first 375×812.
- **Hierarchy:** Define H1/H2/H3 per screen before laying out.

### 2. Produce wireframe.html

Single self-contained HTML file:

- Inline CSS, vanilla JS, zero external dependencies
- Phone frame 375×812 with border-radius and box-shadow for realism
- Screen-to-screen navigation via JS (show/hide divs)
- Left sidebar: screen list + navigation flow
- Right sidebar: PM annotation panel (updates per screen)
- **Lo-Fi palette only:** `#F5F5F5` / `#E0E0E0` / `#C8C8C8` / `#888` / `#333` / `#1A1A1A`
- Zero brand colors, zero gradients, zero decorative shadows
- Dynamic states annotated inline (empty, error, loading, success)

**Mandatory per-screen components:**

- Status bar (44px) with time and signal indicators
- Screen header (app header or back header with ←)
- Scrollable content
- Bottom nav (72px) with 4 items (if app is multi-tab)

### 3. Annotations for 5 audiences (wireframes.md §5)

Right sidebar panel updates per screen. For each screen, cover:

1. **Developers:** state logic, business rules, edge cases
2. **Stakeholders:** business benefit, narrative
3. **Designers:** UX intent, hierarchy
4. **Copywriters:** tone-of-voice notes, CTA wording
5. **Future-self:** rationale behind decisions

### 4. PM Checklist (wireframes.md §8)

Before saving, confirm all 10 items pass:

1. Fidelity matches current uncertainty
2. Flow drawn as storyboard, not isolated screens
3. Hierarchy reflects user/business priorities
4. Accessibility: H1-H3 defined, descriptive links
5. Annotations cover all 5 audiences per screen
6. Dynamic states covered (error, loading, empty, success)
7. 8-point grid applied
8. Real content (no Lorem Ipsum)
9. No dark patterns or cognitive asymmetries
10. Each element explains benefit, not just function

### 5. Report

After saving, summarize in chat:

- Screens produced (count + names)
- Entities reflected
- Dynamic states covered
- Open questions (if any) for the user before Designer phase

## Outputs

| File | Description |
|------|-------------|
| `wireframes/wireframe.html` | Self-contained navigable Lo-Fi wireframe with annotations |

## Notes

- Never use brand colors or visual polish — forces strategic feedback.
- Never produce React/Vue components — output is always pure HTML.
- Never jump to Hi-Fi even if requested — explain why Lo-Fi is right at this stage.
- Full methodology reference: `.claude/skills/generate-wireframes/wireframes.md`.

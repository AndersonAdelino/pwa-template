---
name: pm
description: Phase 1 of the PWA pipeline. Acts as Senior Product Manager — writes a PRD, then a LOW-FIDELITY navigable wireframe (Balsamiq-style sketch, no colors/typography polish). Use when starting Phase 1, planning UX structure, writing requirements, or building the wireframe. Triggers on "fase 1", "pm", "PRD", "wireframe", "requisitos".
---

# Skill: pm — Product Manager (Phase 1)

Self-contained Phase 1. Produces `prd.md` then `wireframes/wireframe.html` (true lo-fi sketch). Stop after each artifact for user approval.

## Output language

All user-facing text (chat, PRD, wireframe annotations, PROJECT.md updates) in Brazilian Portuguese (pt-BR). File paths, CSS/HTML/JS identifiers, technical terms (RLS, PWA, CRUD) stay in English.

## Context to load first

1. `PROJECT.md` — prior context, decisions, project name
2. `prd.md` if it exists — edit mode
3. `wireframes/wireframe.html` if it exists — iteration mode
4. `.claude/skills/pm/wireframes.md` — methodology reference (read before producing wireframe)

If Phase 1 already done, ask user whether to iterate or restart.

---

## Step 1 — Write PRD

Produces `prd.md` at project root.

### 1.1 Discovery

Ask only what is missing — never re-ask what is already in PROJECT.md or the user brief. Minimum to collect:

- **Problema:** What pain does this app solve? Who has it today?
- **Usuário-alvo:** Persona — quem, contexto, frequência de uso.
- **Solução:** What does the app do in one sentence?
- **Sucesso:** 1-3 measurable signals.

### 1.2 Derive entities

From problem + solution, list domain "nouns" the user creates/reads/updates/deletes. Drives wireframe (forms, cards) and Supabase schema. Be explicit.

### 1.3 User stories

Format: `Como [persona], quero [ação] para que [benefício].`

Golden path first (3-5 stories), then secondary (auth, settings, errors). Each story testable.

### 1.4 Acceptance criteria per story

2-4 criteria per story in Given/When/Then. Become QA test cases.

### 1.5 Out of scope

Explicit list of what the app does NOT do.

### 1.6 PRD template (`prd.md`)

```markdown
# PRD — [Nome do app]

## 1. Visão
**Problema:** ...
**Usuário-alvo:** ...
**Solução em uma frase:** ...
**Métricas de sucesso:**
- ...

## 2. Entidades do domínio
- **[Entidade]:** [descrição curta + campos esperados]
- ...

## 3. User Stories

### US-01 — [Título]
Como [persona], quero [ação] para que [benefício].

**Critérios de aceitação:**
- Given [contexto], when [ação], then [resultado].
- ...

### US-02 — ...

## 4. Telas previstas
- Login / Cadastro
- [Tela principal]
- [Tela CRUD por entidade]
- ...

## 5. Restrições técnicas
- Stack: Next.js 14 + Supabase + Tailwind + PWA
- Deploy: Vercel
- Auth: Supabase Auth (email + senha)
- RLS obrigatório em todas as tabelas

## 6. Fora de escopo (v1)
- ...
```

### 1.7 PRD checklist

- [ ] Problem statement names real user + real pain
- [ ] Success metrics measurable
- [ ] 3+ user stories cover golden path
- [ ] Each story has Given/When/Then criteria
- [ ] Entities listed match stories
- [ ] Out-of-scope non-empty

### 1.8 Stop and report

Show summary in chat. **Wait for explicit user approval before moving to wireframe.**

---

## Step 2 — Lo-Fi Wireframe

Produces `wireframes/wireframe.html` — single self-contained HTML, navigable, **Balsamiq-style sketch** (zero polish).

### 2.1 Apply methodology

Read `.claude/skills/pm/wireframes.md` before producing. Defines fidelity strategy, annotation patterns, accessibility, 10-item checklist.

Key rules:
- **Fidelity:** Lo-Fi mandatory at this phase.
- **Storyboard:** full screen flow, not isolated screens.
- **Real content:** never Lorem Ipsum — use real data from app's domain.
- **Hierarchy:** define H1/H2/H3 per screen before laying out.

### 2.2 Build wireframe.html — TRUE LO-FI / BALSAMIQ STYLE

Single self-contained HTML, inline CSS, vanilla JS, zero external deps **except one handwritten Google Font** for sketch feel.

**Mandatory visual rules (no exceptions):**

- **Font:** `Patrick Hand` OR `Caveat` from Google Fonts — applied to ALL text in the wireframe area. Sidebar UI may use system font.
- **Palette:** GRAYSCALE ONLY. Allowed colors: `#FFFFFF`, `#F5F5F5`, `#E0E0E0`, `#BBBBBB`, `#666666`, `#222222`. No blue/red/green/yellow/brand colors anywhere in the wireframe (annotations sidebar can use one accent for numbered callouts).
- **Borders:** `border: 2px dashed #666` or `border: 2px solid #222` — sketchy. Never thin smooth borders.
- **Backgrounds:** white or light gray fills only. No gradients. No shadows on wireframe elements (phone frame shadow OK).
- **Images:** rectangle with diagonal lines `linear-gradient(45deg, #E0E0E0 25%, transparent 25%, transparent 75%, #E0E0E0 75%)` or simple box with label `[IMG]` / `[FOTO]`. Never real images.
- **Icons:** text labels (`[search]`, `[+]`, `[menu]`, `[<]`) OR rough hand-drawn SVG. Never polished icon sets.
- **Buttons:** dashed/solid box with handwritten label. No rounded brand buttons, no fills with brand colors.
- **Inputs:** underline or dashed box. Placeholder in italic gray.
- **No CSS animations except simple opacity/transform for screen navigation.**

**Layout:**

- Phone frame 375×812 with border-radius and one neutral box-shadow (frame realism is fine — content inside stays sketchy)
- Screen-to-screen navigation via JS (show/hide divs)
- Left sidebar: screen list + flow arrows
- Right sidebar: PM annotation panel (updates per screen)
- Dynamic states inline-annotated (empty, error, loading, success)

**Mandatory per-screen components:**

- Status bar (44px) — just text `09:41 ··· · ▮▮▮` in handwritten font
- Screen header (app header or back header with `[<]`)
- Scrollable content area
- Bottom nav (72px) with text labels if multi-tab app

### 2.3 Annotations for 5 audiences (right sidebar, updates per screen)

1. **Desenvolvedores:** state logic, business rules, edge cases
2. **Stakeholders:** business benefit, narrative
3. **Designers:** UX intent, hierarchy (NOT colors/fonts — that's Phase 2)
4. **Copywriters:** tone-of-voice, CTA wording
5. **Eu do futuro:** rationale behind decisions

Numbered callouts in the wireframe link to annotations. One accent color allowed on the numbered circles only (e.g. `#D32F2F` for visibility).

### 2.4 PM Checklist (before saving)

Confirm all 10:

1. Fidelity matches uncertainty (Lo-Fi for discovery)
2. Flow as storyboard, not isolated screens
3. Hierarchy reflects user/business priority
4. Accessibility: H1-H3 defined, descriptive links
5. Annotations cover all 5 audiences per screen
6. Dynamic states covered (error, loading, empty, success)
7. 8-point grid applied
8. Real content (no Lorem Ipsum)
9. No dark patterns
10. Each element explains benefit, not function

### 2.5 Report and handoff

After saving, summarize in chat:
- Screens produced (count + names)
- Entities reflected
- Dynamic states covered
- Open questions for user before Phase 2

Update `PROJECT.md` Phase 1 section with: PRD summary, wireframe path, key decisions, handoff notes for Designer.

Suggest next step: invoke `/designer` skill for Phase 2.

---

## Outputs

| File | Description |
|------|-------------|
| `prd.md` | Product requirements (project root) |
| `wireframes/wireframe.html` | Self-contained lo-fi navigable wireframe |

## Hard rules

- Never skip PRD step — Hi-Fi never allowed in Phase 1.
- Never invent screens/features absent from user brief.
- Real content only — no Lorem Ipsum.
- Annotate for 5 audiences per screen.
- **Wireframe is a SKETCH. If a stakeholder could mistake it for a finished design, you went too far. Force them to discuss structure, not aesthetics.**
- Stop after PRD for approval. Stop after wireframe for approval.

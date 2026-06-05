---
name: designer
description: Phase 2 of the PWA pipeline. Acts as Senior UI/UX Designer — converts the approved Lo-Fi wireframe into a Hi-Fi design with an approved design system. Use when starting Phase 2, designing the UI, or building the Hi-Fi mockup. Triggers on "fase 2", "designer", "UI", "UX", "design", "Hi-Fi".
---

# Skill: designer — UI/UX Designer (Phase 2)

Self-contained Phase 2. Derives a domain-appropriate design system, proposes it for approval, then builds `designs/design.html`.

## Output language

Design system proposal, annotations, chat updates in Brazilian Portuguese (pt-BR). CSS tokens, font names, hex codes stay verbatim.

## Context to load

1. `PROJECT.md` — app type, target audience, PM decisions
2. `prd.md` — solution, entities, success metrics
3. `wireframes/wireframe.html` — structure, screens, flow, PM annotations
4. `designs/design.html` if it exists — iteration mode

Phase 1 must be done. If not, instruct user to run `pm` skill first.

## Process

### 1. Extract requirements from wireframe

Before any visual decision, map:

- All screens and components present
- Visual hierarchy defined by PM (urgent / primary / secondary)
- Annotated dynamic states (empty, error, loading, success)
- Reusable components identified in annotations
- Navigation flow and interaction patterns

### 2. Propose Design System

**If user provided style preferences in arguments:** use directly, skip proposal.

**Otherwise:** analyze context and present proposal for approval.

Derive design system from:
- App type/domain (health, finance, productivity, etc.)
- Target audience inferred from PROJECT.md + wireframe
- Tone/mood communicated by PM annotations
- Established visual patterns for the domain

Present proposal in chat using this exact format, then **wait for explicit user approval before writing any file**:

```markdown
## Proposta de Design System — [nome do projeto]

**Mood:** [3 palavras-chave]
**Justificativa:** [por que essas escolhas fazem sentido para esse domínio e público]

**Paleta:**
- Primária: #XXXXXX — [papel: ações principais, CTAs]
- Secundária: #XXXXXX — [papel: ações secundárias, destaques]
- Warning: #XXXXXX — [alertas]
- Danger: #XXXXXX — [erros, ações destrutivas]
- Success: #XXXXXX — [confirmações]
- Background: #XXXXXX
- Surface (cards/painéis): #XXXXXX
- Texto principal: #XXXXXX
- Texto secundário: #XXXXXX
- Borda: #XXXXXX

**Tipografia:**
- Família: [fonte] — [justificativa]
- H1: Xpx / peso 700
- H2: Xpx / peso 600
- Body: Xpx / peso 400
- Caption: Xpx / peso 400

**Spacing:** sistema [X]pt (múltiplos de [X]px)
**Border radius:** Xpx cards · Xpx inputs · Xpx chips · 50% FAB
**Elevation:** [descrição das sombras por nível]

Aprovar para construir, ou ajustar algum ponto antes de continuar?
```

### 3. Build designs/design.html

After approval, single self-contained HTML file:

- `<link>` to Google Fonts in `<head>` for approved typeface
- CSS custom properties in `:root` for every design system token
- Same structure as wireframe: left sidebar (flow) + phone frame + right sidebar (annotations)
- Same dimensions as wireframe (e.g. 375×812)
- JS screen navigation matching wireframe pattern

Per component:
- Final visual style using approved tokens
- Inline SVG icons (no external CDN)
- Visual states: hover, focus, disabled, checked, loading
- Micro-interactions via CSS `transition` (no heavy JS)
- Semantic status indicators using approved palette

Components: derived from wireframe annotations (whatever was marked as reusable).

### 4. Design annotations (right sidebar)

Per screen, document for Engineer:

- CSS tokens applied to main elements
- Reusable component name
- Expected interactions (hover / tap / swipe / focus)
- Accessibility: WCAG AA contrast verified, required aria-labels
- Component variants (e.g. chip: ok | low | critical)

### 5. Report and handoff

After saving, summarize in chat:
- Tokens registered (primary color, font family, spacing scale)
- Components implemented (count + names with variants)
- Accessibility status (contrast pass/fail per token pair)

Update `PROJECT.md` Phase 2 section with: design system summary, design.html path, handoff notes for Engineer.

Suggest next step: invoke `engineer` skill for Phase 3.

## Outputs

| File | Description |
|------|-------------|
| `designs/design.html` | Hi-Fi navigable design with approved system |

## Hard rules

- No CSS frameworks in design.html — plain CSS with custom properties.
- No React components — design.html is a visual reference for Engineer.
- Same screens and flow as wireframe — never invent new screens.
- Inline SVG icons only — never external CDN.
- Validate WCAG AA contrast: 4.5:1 body text, 3:1 large text.
- Wait for explicit user approval on the design system proposal before generating the HTML.

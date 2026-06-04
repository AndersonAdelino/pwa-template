---
name: create-ui-design
description: Generates a Hi-Fi UI design HTML file from an approved Lo-Fi wireframe. Derives a domain-appropriate design system, proposes it for explicit approval, then builds the polished design.html. Use when a UI/UX Designer needs to convert the wireframe into a production-ready visual reference for the Engineer phase.
---

# Skill: create-ui-design

Produces `designs/design.html` — a single self-contained HTML file with the Hi-Fi design, full design system tokens, and engineer-facing annotations.

## Output language

Design system proposal, annotations, and chat updates are in Brazilian Portuguese (pt-BR). CSS tokens, font names, and hex codes stay in their native format.

## Context to load

1. `PROJECT.md` — app type, target audience, PM decisions
2. `prd.md` — solution, entities, success metrics
3. `wireframes/wireframe.html` — structure, screens, flow, PM annotations
4. `designs/design.html` if it exists — iteration mode

## Process

### 1. Extract requirements from wireframe

Before any visual decision, map:

- All screens and components present in the wireframe
- Visual hierarchy defined by the PM (urgent / primary / secondary)
- Annotated dynamic states (empty, error, loading, success)
- Reusable components identified in annotations
- Navigation flow and interaction patterns

### 2. Propose Design System (hybrid mode)

**If $ARGUMENTS contains style preferences:** use directly, skip proposal.

**Otherwise:** analyze context and present proposal for approval.

Derive the design system from:

- App type and domain (e.g. health, finance, productivity, cooking)
- Target audience inferred from PROJECT.md and wireframe
- Tone/mood communicated by PM annotations
- Established visual patterns for the domain

Present the proposal in chat using this exact format, then **wait for explicit user approval before writing any file**:

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

After approval:

Single self-contained HTML file:

- `<link>` to Google Fonts in `<head>` for the approved typeface
- CSS custom properties in `:root` for every design system token
- Same structure as the wireframe: left sidebar (flow) + phone frame + right sidebar (annotations)
- Same dimensions as the wireframe (e.g. 375×812)
- JS screen navigation matching the wireframe pattern

Per component, implement:

- Final visual style using approved tokens
- Inline SVG icons (no external CDN)
- Visual states: hover, focus, disabled, checked, loading
- Micro-interactions via CSS `transition` (no heavy JS)
- Semantic status indicators using the approved palette

Components to implement: derived from wireframe annotations (whatever was marked as reusable).

### 4. Design annotations (right sidebar)

Per screen, document for the Engineer:

- CSS tokens applied to main elements
- Reusable component name
- Expected interactions (hover / tap / swipe / focus)
- Accessibility notes: WCAG AA contrast verified, required aria-labels
- Component variants (e.g. chip: ok | low | critical)

### 5. Report

After saving, summarize in chat:

- Tokens registered (primary color, font family, spacing scale)
- Components implemented (count + names with variants)
- Accessibility check status (contrast pass/fail per token pair)

## Outputs

| File | Description |
|------|-------------|
| `designs/design.html` | Hi-Fi navigable design with approved design system |

## Notes

- No CSS frameworks in design.html — plain CSS with custom properties.
- No React components — design.html is a visual reference for the Engineer.
- Same screens and flow as the wireframe — never invent new screens.
- Inline SVG icons only — never an external CDN dependency.
- Validate WCAG AA contrast: 4.5:1 for body text, 3:1 for large text.

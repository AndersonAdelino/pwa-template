---
name: write-prd
description: Writes a Product Requirements Document (prd.md) for a PWA app. Use when a Product Manager needs to formalize problem statement, target users, user stories, success metrics, and acceptance criteria before any UI work.
---

# Skill: write-prd

Produces `prd.md` at project root — a lean PRD that captures everything the wireframe, design, and engineer downstream need to derive their work from.

## Output language

All PRD content is written in Brazilian Portuguese (pt-BR). Section headings, technical terms (RLS, PWA, CRUD), and identifiers stay in English when conventional.

## Input expected

- App name (from chat or PROJECT.md)
- Short description / problem the app solves
- Target user persona (if provided)
- Constraints (optional: platform, integrations, deadline)

If any required input is missing, ask the user before writing.

## Process

### 1. Discovery questions

Ask only what is missing — never re-ask what is already in PROJECT.md or in the user brief. Minimum to collect:

- **Problema:** What pain does this app solve? Who has it today?
- **Usuário-alvo:** Persona principal — quem, qual contexto, qual frequência de uso.
- **Solução:** What does the app do in one sentence?
- **Sucesso:** How will we know the app worked? (1-3 measurable signals)

### 2. Derive entities

From the problem and solution, list the domain "nouns" the user creates / reads / updates / deletes. These will drive both the wireframe (forms, cards) and the Supabase schema. Be explicit even if obvious.

### 3. Write user stories

Format: `Como [persona], quero [ação] para que [benefício].`

Cover the golden path first (3-5 stories), then secondary flows (auth, settings, errors). Each story must be testable.

### 4. Acceptance criteria per story

For each story, write 2-4 acceptance criteria in Given/When/Then format. These become QA's test cases later.

### 5. Out of scope

Explicitly list what this app does NOT do, to prevent scope creep in Design / Engineer phases.

## Output template (`prd.md`)

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

## Quality checklist

Before saving the file, confirm:

- [ ] Problem statement names a real user and a real pain
- [ ] Success metrics are measurable (not vague)
- [ ] At least 3 user stories cover the golden path
- [ ] Each story has acceptance criteria in Given/When/Then
- [ ] Entities listed match what the user stories create/edit
- [ ] Out-of-scope section is non-empty

## Outputs

| File | Description |
|------|-------------|
| `prd.md` | Product requirements document at project root |

## Notes

- Keep it lean — PRD should fit on 1-2 pages printed. Long PRDs don't get read.
- Reference real product examples in domain — gives wireframe + design something concrete to anchor to.
- Stop after writing. Show summary in chat and ask user to approve before agent moves to wireframe.

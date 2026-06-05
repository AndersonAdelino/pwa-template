# PWA Template — Pipeline de 5 Fases (Skills-only)

Template reutilizável para apps PWA com Next.js + Supabase + Vercel. Copie esta pasta para qualquer projeto novo e ajuste apenas `PROJECT.md` + `.mcp.json` + `.env.example`.

## Projeto

**Nome:** _[preencher após Fase 1]_
**Stack:** Next.js 14 · Supabase · Tailwind · Vercel · PWA

## Arquitetura — apenas Skills

Sem subagents, sem slash commands. Cada fase = 1 Skill auto-contida. Você (orquestrador, neste chat) invoca a Skill quando o usuário pedir aquela fase. Fluxo recomendado: **uma sessão separada por fase** para economizar contexto/tokens.

```
Orchestrator (eu, neste chat)
├── Skill `pm`       → Fase 1: PRD + wireframe lo-fi
├── Skill `designer` → Fase 2: Hi-Fi design + design system
├── Skill `engineer` → Fase 3: Supabase + Next.js + PWA
├── Skill `qa`       → Fase 4: testes em browser real (Chrome DevTools MCP)
└── Skill `security` → Fase 5: review + OWASP + deploy GitHub + Vercel
```

## Pipeline

| Fase | Skill | O que produz |
|------|-------|--------------|
| 1 — Product Manager | `pm` | `prd.md` + `wireframes/wireframe.html` (lo-fi sketch) |
| 2 — UI/UX Designer | `designer` | `designs/design.html` (hi-fi + design system aprovado) |
| 3 — Software Engineer | `engineer` | Supabase backend + `src/` (Next.js + Tailwind + PWA) |
| 4 — QA Engineer | `qa` | App validado, bugs corrigidos, relatório em `PROJECT.md` |
| 5 — Security Engineer | `security` | Auditoria OWASP + repo GitHub privado + deploy Vercel |

## Regras Globais

- Sempre ler `PROJECT.md` antes de agir em qualquer fase
- Saídas ao usuário em pt-BR — código, identificadores, paths em inglês
- Commits e repositórios via `gh` CLI; privados por padrão (regra global em `~/.claude/CLAUDE.md`)
- TypeScript strict em todo código de produção
- Cada Skill atualiza `PROJECT.md` ao concluir sua fase
- `tsc --noEmit` e `eslint --max-warnings 0` obrigatórios antes de qualquer commit ou avanço de fase
- **Sessões separadas por fase** para reduzir consumo de tokens — abra novo chat ao passar de fase

## Artefatos Gerados por Fase

- Fase 1: `prd.md`, `wireframes/wireframe.html`
- Fase 2: `designs/design.html`
- Fase 3: `src/` (Next.js app), `supabase/migrations/`, `.env.local`, `.env.example`
- Fase 4: app validado, bugs corrigidos, relatório em `PROJECT.md`
- Fase 5: auditoria OWASP, repo GitHub privado, deploy Vercel, URL de produção em `PROJECT.md`

## Contexto Técnico

_[preencher conforme o projeto avança]_

- **Supabase Project URL:** _[via MCP — `mcp__supabase__get_project_url`]_
- **Supabase Anon Key:** ver `.env.local` (não commitado)
- **Vercel URL:** _[preenchido na Fase 5]_
- **GitHub repo:** _[preenchido na Fase 5]_

## Setup inicial em um novo projeto

1. Copiar `.claude/`, `CLAUDE.md`, `PROJECT.md`, `.gitignore`, `.env.example`, `.mcp.json` para a nova pasta
2. Editar `.mcp.json` — substituir `<SUPABASE_PROJECT_REF>` pelo project_ref real do Supabase
3. `git init && git add . && git commit -m "init pwa template"`
4. Em um chat novo, pedir: "começa a Fase 1: [descrição do app]" — o orquestrador invoca a Skill `pm`

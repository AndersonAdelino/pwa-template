# PWA Template — Pipeline de 5 Fases

Template reutilizável para apps PWA com Next.js + Supabase + Vercel. Copie esta pasta para qualquer projeto novo e ajuste apenas `PROJECT.md` + `.mcp.json` + `.env.example`.

## Projeto

**Nome:** _[preencher após Fase 1]_
**Stack:** Next.js 14 · Supabase · Tailwind · Vercel · PWA

## Pipeline

| Fase | Slash command | Subagent | Skills invocadas |
|------|---------------|----------|------------------|
| 1 — Product Manager | `/pm` | `product-manager` | `write-prd` → `generate-wireframes` |
| 2 — UI/UX Designer | `/designer` | `designer` | `create-ui-design` |
| 3 — Software Engineer | `/engineer` | `engineer` | `setup-supabase` → `implement-frontend` |
| 4 — QA Engineer | `/qa` | `qa` | `run-qa-tests` |
| 5 — Security Engineer | `/security` | `security` | `review-code` → `security-audit` → `deploy-vercel` |

## Regras Globais

- Sempre ler `PROJECT.md` antes de agir em qualquer fase
- Saídas ao usuário em português brasileiro (pt-BR) — código, identificadores, paths em inglês
- Commits e repositórios via `gh` CLI; privados por padrão (regra global do usuário em `~/.claude/CLAUDE.md`)
- TypeScript strict em todo código de produção
- Cada subagent atualiza `PROJECT.md` ao concluir sua fase
- `tsc --noEmit` e `eslint --max-warnings 0` obrigatórios antes de qualquer commit ou avanço de fase

## Arquitetura — Orchestrator, Agents, Skills

- **Orchestrator:** este chat (Claude principal). Lê `PROJECT.md`, identifica fase atual, sugere ou executa o slash command correto.
- **Subagents (`.claude/agents/`):** 5 papéis especializados, contexto isolado, modelo Sonnet 4.6. Invocados via slash commands ou pelo Agent tool.
- **Skills (`.claude/skills/`):** 9 ferramentas atômicas, invocadas pelos subagents via Skill tool.

```
Orchestrator (eu)
├── /pm → product-manager → write-prd, generate-wireframes
├── /designer → designer → create-ui-design
├── /engineer → engineer → setup-supabase, implement-frontend
├── /qa → qa → run-qa-tests
└── /security → security → review-code, security-audit, deploy-vercel
```

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
4. Rodar `/pm [descrição do app]` para iniciar a Fase 1

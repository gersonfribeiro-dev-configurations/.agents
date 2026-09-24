---
name: gitlens-mcp
description: Use when managing Git history, worktrees, branches, analyzing diffs, writing commits, or interacting with issues and Pull Requests via the GitKraken/GitLens MCP.
---

# GitLens/GitKraken: rastreabilidade da entrega

## Responsabilidades

- Ler status/diff/log/blame da branch e contexto de issue/PR com GitLens e ferramentas GitHub disponíveis; não presumir que o MCP sempre expõe ou sempre omite campos de Project. Confirmar suporte atual e usar GitHub CLI/GraphQL quando necessário.
- Confirmar `gh auth status`, identidade com acesso ao Project e remoto SSH antes de push. SSH transporta commits, não altera metadados.
- Conferir milestone e épica reais, status, bloqueios e destino da release antes de criar a branch. Uma sub-issue = uma branch e um worktree partindo de `release/<milestone>` (criada de `develop` se faltar). Hotfix pode partir do destino publicado; nunca fixar `release/v0.0.1`.

## Commits, PRs e integração

- Commits seguem Conventional Commits e referenciam a issue sem fechá-la antecipadamente. Uma branch de entrega gera um PR de sub-issue; PRs de integração `release → develop → master` agregam entregas após homologação de cada etapa.
- Antes de abrir PR, revisar diff e metadados da issue; solicitar review humano do responsável do repositório, nunca autoaprovar.

## Status

Consultar o Project real e seus workflows. `Prevented`: issue necessária, mas bloqueada; `On hold`: fila de próximas iterações, inclusive desbloqueantes. `In progress`: trabalho iniciado; `In review`: PR sob revisão; `Request changes`: mudanças solicitadas. `Ready`: PR aceito e reservado para a release, **aguardando merge**. `Done`: merge/fechamento. Não assumir que um evento faz a automação disparar sem verificá-la; conferir vínculos e corrigir com a API apenas quando necessário.

## Checklist

- [ ] Branch, worktree e remoto conferidos com status/diff do Git?
- [ ] Milestone, bloqueios e Project consultados por nome/IDs reais?
- [ ] Campos usados pela ferramenta escolhida existem e têm as opções esperadas?
- [ ] PR vinculado à issue correta, revisão/homologação e Status antes do merge confirmados?

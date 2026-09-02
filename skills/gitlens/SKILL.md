---
name: gitlens-mcp
description: Use when managing Git history, worktrees, branches, analyzing diffs, writing commits, or interacting with issues and Pull Requests via the GitKraken/GitLens MCP.
---
# Skill: Gestao de Repositorio (GitLens MCP) - Alinhada ao Fluxo Release

## 1. Papel

Atue como Engenheiro de Integracao. Orquestre historico com rastreabilidade total via MCP GitKraken/GitLens. Zero suposicoes: leia `git_status`, `git_log_or_diff`, `git_blame` antes de qualquer proposta.

## 2. Triagem integrada

Use `issues_assigned_to_me`, `gitlens_launchpad` e `gh issue view` para carregar a epica `v0.0.1`, suas sub-issues, Milestone e bloqueios antes de iniciar. Valide Project `Status`, `Estimate`, `Size`.

## 3. Worktrees e Branches (obrigatorio)

- Uma issue = um worktree a partir de `release/v0.0.1` (ver `git-worktree`). Nunca reutilizar worktree entre issues.
- Nomenclatura Git Flow estrita: `feature/` para `Feature/Bug/Task`, `hotfix/` so para `Hotfix`, `release/v0.0.1` para preparacao de versao. Nunca `fix/`.
- Branches de issue voltam para `release/v0.0.1` via PR. `release/*` volta para `develop`; `develop` volta para `master`.

## 4. Commits

Use `gitlens_commit_composer`. Mensagens `Conventional Commits` (`feat:`, `fix:`...) com corpo explicando motivo e `Refs #N` da issue. O `Closed #N` sofecha automaticamente em PR para `master`.

## 5. Status do Project

Respeitar transicoes: `In Progress` (branch criada), `In Review` (PR aberto), `Ready` (APPROVED do owner), `Done` (merge). Nao forcar manualmente se workflow cobre; diagnosticar ausencia de item no Project ou vinculo errado primeiro.

## 6. Checklist

- [ ] `git_status`/`diff` lidos via MCP?
- [ ] Branch parte de `release/v0.0.1` (ou `master` se Hotfix)?
- [ ] Worktree isolado criado?
- [ ] Commit segue Conventional Commits e referencia a issue?
- [ ] PR com mesmos metadados da issue e review do owner solicitado?

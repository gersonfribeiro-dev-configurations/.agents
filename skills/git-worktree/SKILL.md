---
name: git-worktree
description: Use when creating, listing, navigating, moving, locking, removing, pruning, or diagnosing Git worktrees and branches already checked out in another worktree.
---

# Skill: Git Worktree - Fluxo Obrigatorio com Release Branch

Use worktree SEMPRE que for entregar uma issue vinculada a uma Milestone/Release. O worktree e parte do fluxo oficial, nao opcional.

## Limite de responsabilidade

Worktree isola apenas o codigo e o historico Git. A issue, o PR e os metadados do Project continuam sendo recursos remotos: use MCP para operacoes de alto nivel e `gh api graphql` para fields/views do Project V2. Use SSH somente no remote Git para `fetch`, `pull` e `push`.

## Principios

- Um worktree = uma delivery = uma issue/sub-issue da epica `v0.0.1`.
- A branch da issue parte obrigatoriamente da **branch de release** (`release/v0.0.1`). Se a release nao existe, cria-la a partir de `develop` primeiro: `git checkout -b release/v0.0.1 develop && git push -u origin release/v0.0.1`.
- Nunca partir de `master` para `feature/bug/task`. `hotfix/` parte de `master` ou `release/*` publicada.
- Uma branch so pode estar ativa em um worktree por vez. Navegue ate o worktree em vez de `git checkout` em outro diretorio.
- Nunca apague a pasta manualmente. Use `git worktree remove`.

## Fluxo padrao para uma sub-issue

```bash
# 1. Garantir que a release branch existe e esta atualizada
git fetch origin
git checkout release/v0.0.1
git pull --ff-only

# 2. Criar worktree + branch da issue a partir da release
# <caminho-do-worktree> deve ser relativo ao repo ou usar variavel de ambiente, nunca caminho absoluto com usuario fixo
# Exemplo relativo (recomendado, funciona em qualquer maquina): ../<repo>-<slug>
# Ou com base configuravel: $WORKSPACE/<repo>-<slug>
git worktree add -b feature/<slug-da-issue> ../<repo>-<slug> release/v0.0.1

# 3. Trabalhar dentro do worktree
cd ../<repo>-<slug>
git status

# 4. Publicar branch para PR voltar para release
git push -u origin feature/<slug-da-issue>
```

Ao criar a branch, adicionar a issue ao Project e mover `Status` para `In Progress` (workflow ou manual se permissao faltar e registrado em comentario). Adicionar item e atualizar fields devem ser feitos via GraphQL quando o MCP nao expuser a operacao.

## Consultar worktrees

```bash
git worktree list
git worktree list --porcelain
git -C <caminho-do-worktree> status --short
```

## Navegar

Nao existe comando Git para trocar diretorio do terminal. Use `cd <caminho-do-worktree>`.

## Branch ja usada por outro worktree

```bash
git worktree list
cd <caminho-informado-pelo-git>
# ou remova se nao precisa mais
git -C <caminho> status --short
git worktree remove <caminho>
```

## Mover, proteger e remover

```bash
git worktree move <atual> <novo>
git worktree lock --reason "Em uso externo" <caminho>
git worktree unlock <caminho>
git worktree remove <caminho>
```

## Limpar registros obsoletos

```bash
git worktree prune --dry-run
git worktree prune
```

## Diagnostico rapido

```bash
git branch --show-current
git rev-parse --show-toplevel
git worktree list
```

## Regra de ouro

Sem worktree separado, sem entrega. O diretorio principal permanece em `develop`/`master`; cada `feature/*` vive em seu worktree irmao (ex: `../<repo>-<slug>`) ate merge e `Done`. Nunca usar caminho absoluto com `C:/Users/<usuario>` fixo - use caminho relativo ou variavel (`$HOME`, `%USERPROFILE%`, `$WORKSPACE`).

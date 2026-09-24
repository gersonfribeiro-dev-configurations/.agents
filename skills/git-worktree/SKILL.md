---
name: git-worktree
description: Use when creating, listing, navigating, moving, locking, removing, pruning, or diagnosing Git worktrees and branches already checked out in another worktree.
---

# Skill: Git Worktree - Fluxo Obrigatorio com Release Branch

Use worktree SEMPRE que for entregar uma issue vinculada a uma Milestone/Release. O worktree e parte do fluxo oficial, nao opcional.

## Limite de responsabilidade

Worktree isola apenas o código e o histórico Git. Issue, PR e metadados do Project continuam remotos: use MCP/GitHub CLI nas operações suportadas e GraphQL quando necessário; SSH somente para transporte Git.

## Principios

- Um worktree = uma entrega = uma issue/sub-issue da épica da milestone efetiva.
- A branch da sub-issue parte de `release/<milestone>`; se não existe, criá-la a partir de `develop` depois de conferir o nome da milestone e o remoto. Não usar versão fixa nos comandos.
- Nunca partir de `master` para `feature/bug/task`. `hotfix/` parte de `master` ou `release/*` publicada.
- Uma branch so pode estar ativa em um worktree por vez. Navegue ate o worktree em vez de `git checkout` em outro diretorio.
- Nunca apague a pasta manualmente. Use `git worktree remove`.

## Fluxo padrao para uma sub-issue

```bash
# 1. Substituir os valores abaixo pelos nomes reais da milestone, issue e repositório
MILESTONE=v1.0.0
SLUG=nome-da-issue
REPO=nome-do-repositorio
# Garantir que a release branch já existe no remoto (se não existir, criá-la de develop)
git fetch origin

# 2. Criar worktree + branch da issue a partir da release
# Usar caminho relativo ou base configurável, nunca usuário absoluto fixo
git worktree add -b "feature/${SLUG}" "../${REPO}-${SLUG}" "origin/release/${MILESTONE}"

# 3. Trabalhar dentro do worktree
cd "../${REPO}-${SLUG}"
git status

# 4. Publicar branch para PR voltar para release
git push -u origin "feature/${SLUG}"
```

Ao criar a branch, verificar se a issue está no Project e se o workflow moveu `Status` para `In progress` (grafia oficial). Se não, diagnosticar vínculo e automação antes de atualizar manualmente. MCP, CLI e GraphQL podem operar fields conforme cobertura real.

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

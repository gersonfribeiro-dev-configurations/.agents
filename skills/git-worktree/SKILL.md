---
name: git-worktree
description: Use when creating, listing, navigating, moving, locking, removing, pruning, or diagnosing Git worktrees and branches already checked out in another worktree.
---

# Skill: Git Worktree

Use esta skill para administrar múltiplos diretórios de trabalho de um mesmo repositório Git. Um worktree permite trabalhar em branches ou commits distintos simultaneamente, sem alternar a branch do diretório principal.

## Princípios

- Um worktree vinculado possui seu próprio diretório de trabalho, mas compartilha o mesmo repositório Git e seus objetos.
- Uma branch só pode estar ativa em um worktree por vez. Para trabalhar nessa branch, navegue até seu worktree em vez de executar `git checkout` em outro.
- Não exclua manualmente a pasta de um worktree. Use `git worktree remove` para remover tanto os arquivos quanto o registro administrativo.
- Antes de remover um worktree, verifique alterações pendentes com `git -C <caminho> status --short`.
- Não use `--force` para remover worktrees com alterações sem confirmar explicitamente que o descarte é desejado.

## Consultar worktrees

Liste os worktrees registrados no repositório atual:

```bash
git worktree list
```

Use o formato legível por ferramentas para diagnósticos ou scripts:

```bash
git worktree list --porcelain
```

Consulte o estado de todos os worktrees, inclusive arquivos modificados:

```bash
git worktree list --porcelain
git -C <caminho-do-worktree> status --short
```

## Navegar entre worktrees

Não existe um comando Git para trocar o diretório atual do terminal. Navegue normalmente até o diretório que representa o worktree desejado:

```bash
cd <caminho-do-worktree>
git status
```

Em Git Bash, para abrir o worktree que está ao lado do repositório atual:

```bash
cd ../nome-do-worktree
```

## Criar worktrees

Crie um worktree para uma branch existente:

```bash
git worktree add <novo-caminho> <branch-existente>
```

Crie uma branch nova já associada ao novo worktree:

```bash
git worktree add -b <nova-branch> <novo-caminho> [<inicio>]
```

Crie um worktree em um commit específico, em modo detached:

```bash
git worktree add --detach <novo-caminho> <commit-ou-tag>
```

Exemplos:

```bash
git worktree add ../app-correcao feature/correcao
git worktree add -b feature/relatorio ../app-relatorio main
```

## Branch já usada por outro worktree

Quando o Git informar que uma branch já está usada por outro worktree, identifique o diretório com:

```bash
git worktree list
```

Para trabalhar nela, entre no diretório informado:

```bash
cd <caminho-informado-pelo-git>
```

Se o worktree não for mais necessário, valide que não há alterações pendentes e remova-o antes de fazer checkout da branch em outro worktree:

```bash
git -C <caminho-do-worktree> status --short
git worktree remove <caminho-do-worktree>
git switch <branch>
```

## Mover, proteger e remover

Mova um worktree para outro local mantendo seu vínculo com o repositório:

```bash
git worktree move <caminho-atual> <novo-caminho>
```

Evite que um worktree seja removido automaticamente durante uma limpeza:

```bash
git worktree lock --reason "Em uso externo" <caminho-do-worktree>
git worktree unlock <caminho-do-worktree>
```

Remova um worktree que não é mais necessário:

```bash
git worktree remove <caminho-do-worktree>
```

## Limpar registros obsoletos

Se uma pasta de worktree foi apagada fora do Git, primeiro simule a limpeza:

```bash
git worktree prune --dry-run
```

Depois execute a limpeza dos registros obsoletos:

```bash
git worktree prune
```

## Diagnóstico rápido

Para conferir a branch, o diretório atual e os worktrees conhecidos:

```bash
git branch --show-current
git rev-parse --show-toplevel
git worktree list
```

Para consultar ajuda completa da versão local do Git:

```bash
git worktree --help
```

---
name: github-permissions
description: Use when checking PAT scopes, GitHub CLI auth, MCP permissions, or determining which commands must be run by the real user for Projects, Issues, PRs and Packages in the aplicacoesBoilerplate org.
---

# Skill: Permissoes PAT, MCP e GitHub CLI - Matriz de Dependencias

## 1. Principio

O agente usa o PAT/SSH do usuario `agentegersonfribeiro-AI` via `gh` e MCP GitHub. Comandos que exigem `gh auth refresh` ou criacao de recursos com escopo elevado **devem ser executados pelo usuario real** no terminal. O agente nunca tenta adivinhar ou burlar scopes.

## 2. Matriz de scopes por operacao

| Operacao | PAT scope necessario | Ferramenta | Quem executa | Finalidade |
| --- | --- | --- | --- | --- |
| Ler repo, issues, PRs, criar issue/comentario, push branch | `repo` | `gh issue create/edit`, `gh repo view`, `git push`, MCP `GitKraken` | Agente (PAT do agente) | Planejamento e entrega basica |
| Ler/criar Project V2, listar fields, adicionar item ao Project, atualizar `Status/Estimate/Size` | `project` + `read:project` | `gh project list/view/item-add`, `gh api graphql` com `projectsV2` | **Usuario real** via `gh auth refresh -s project,read:project` | O agente recebe `missing required scopes [read:project]` sem isso |
| `write:org` para acoes de org (mudar defaultBranch via API, ajustar settings) | `write:org` | `gh api --method PATCH /repos/...` | **Usuario real** | Agente tem `admin:public_key,gist,read:org,repo` por padrao |
| Solicitar review, criar PR, vincular `Development` | `repo` + `pull-requests: write` (fine-grained) | `gh pr create`, `gh api` | Agente se PAT tiver; senao usuario | `Repository permissions > Pull requests: Read and write` no fine-grained PAT |
| Publicar/ler GitHub Packages (Maven/NPM privado) | `read:packages` + `write:packages` (push) | `mvn deploy`, `npm publish`, `gh auth token` | Agente (CI usa `GITHUB_TOKEN`), dev local via `gh auth login` + `settings.xml`/`~/.npmrc` | `boilerplate auth login` le `gh auth token` |
| Gerenciar `admin:ssh_signing_key` para `gh ssh-key list` | `admin:ssh_signing_key` | `gh ssh-key list` | **Usuario real** | Nao necessario para fluxo normal |

## 3. Comandos que exigem usuario real (copiar exatamente)

```bash
# 1. Conceder Projects (executar uma vez por maquina do agente)
gh auth refresh -s project,read:project -h github.com
# ou completo para org + projects
gh auth refresh -s project,read:project,write:org -h github.com

# 2. Verificar scopes ativos
gh auth status

# 3. Para fine-grained PAT (quando criar PAT no GitHub UI)
# Organization permissions > Projects: Read and write (para editar Status/fields)
# Repository permissions > Issues: Read and write
# Repository permissions > Pull requests: Read and write
# Repository permissions > Contents: Read and write (para push)
# Selecionar apenas repos de aplicacoesBoilerplate

# 4. Para GitHub Packages local (dev)
gh auth login -h github.com   # escolher HTTPS + token com read:packages
cat ~/.m2/settings.xml        # deve ter <server><id>github-boilerplate</id><password>TOKEN</password></server>
cat ~/.npmrc                  # //npm.pkg.github.com/:_authToken=TOKEN

# 5. Mudar defaultBranch quando agente receber 404 em PATCH /repos
gh api --method PATCH /repos/aplicacoesBoilerplate/<repo> -f default_branch="master"
```

## 4. Diagnostico rapido

- `gh: error: your authentication token is missing required scopes [read:project]` -> rodar comando 1 acima.
- `gh: Not Found (HTTP 404) on /orgs/.../projects` -> Projects sao V2, usar `gh project list --owner aplicacoesBoilerplate`, nao REST `/orgs/.../projects`.
- `ERROR: Repository not found. fatal: Could not read from remote` com `git clone git@github.com:...` -> seu `~/.ssh/config` mapeia `Host github.com` para `github-teksystem` (sem acesso privado). Usar alias `git@github-agente.com:` para `agentegersonfribeiro-AI` ou `git@github-pessoal.com:` para `gersonfribeiro`.
- `go vet: missing go.sum entry` -> rodar `docker run --rm -v $(pwd):/app -w /app golang:1.22 go mod tidy` e commitar `go.sum`.
- `illegal character: '\ufeff'` em `javac` -> arquivo com BOM (PowerShell `Set-Content -Encoding UTF8`). Reescrever com `[System.IO.File]::WriteAllText($f, $c, (New-Object System.Text.UTF8Encoding $false))`.
- `go build -o boilerplate ./...` com `cannot write multiple packages` -> usar `go build -o boilerplate .` (package main, nao `./...`).

## 5. O que o agente faz vs o que pede

- **Agente faz sozinho:** criar issues, Milestones, branches, worktrees, commits, pushes, PRs, adicionar itens ao Project quando ja tem `project` scope, `gh run list/view`.
- **Agente pede ao usuario:** conceder scopes (`gh auth refresh`), criar repo com `gh repo create` se org exigir `write:org` adicional, confirmar `defaultBranch`, resolver divergencia de `origin` (fork vs org).

## 6. Verificacao antes de avancar

Antes de criar `release/v0.0.1` ou epic `v0.0.1`, rodar:

```bash
gh auth status                        # confirma project scope
gh project list --owner aplicacoesBoilerplate --format json | jq '.projects[].title'
gh api /repos/aplicacoesBoilerplate/PackagesJava/milestones --jq '.[].title'
git worktree list
git branch -a | grep release
```

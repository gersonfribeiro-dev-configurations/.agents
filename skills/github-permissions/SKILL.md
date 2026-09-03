---
name: github-permissions
description: Use when checking PAT scopes, GitHub CLI auth, MCP permissions, or determining which commands must be run by the real user for Projects, Issues, PRs and Packages across the configured GitHub organizations.
---

# Skill: Permissoes PAT, MCP e GitHub CLI - Matriz de Dependencias

## 1. Principio

O agente usa o PAT do usuario `agentegersonfribeiro-AI` via `gh`/MCP e o alias SSH configurado para Git. Somente a concessao ou renovacao interativa de credenciais (`gh auth refresh`, login, aprovacao de SSO) deve ser executada pelo usuario real. Depois disso, o agente pode operar com o PAT autorizado. Nunca adivinhar ou burlar scopes.

## 2. Matriz de scopes por operacao

| Operacao | PAT scope necessario | Ferramenta | Quem executa | Finalidade |
| --- | --- | --- | --- | --- |
| Ler repo, issues, PRs, criar issue/comentario, push branch | `repo` | `gh issue create/edit`, `gh repo view`, `git push`, MCP `GitKraken` | Agente (PAT do agente) | Planejamento e entrega basica |
| Ler/criar Project V2, listar fields/views, adicionar item ao Project | `project` + `read:project` | `gh project list/view/item-add`, `gh api graphql` com `projectsV2` | Agente, apos autorizacao | O usuario real so executa `gh auth refresh` se o token ainda nao possuir os scopes |
| **Popular/atualizar Custom Fields (`Estimate`, `Size`, `Priority`, `Effort`, `Hotfix`, `Start/Target date`, `Status`)** | `project` + `read:project` | **EXCLUSIVO `gh api graphql` com `updateProjectV2ItemFieldValue`** | Agente (PAT do agente ja com scope) | **MCP NAO cobre ProjectV2 Fields de forma confiavel - usar GraphQL obrigatoriamente.** MCP fica so para labels/assignees/milestone |
| `write:org` para acoes de org (mudar defaultBranch via API, ajustar settings) | `write:org` | `gh api --method PATCH /repos/...` | Agente, se PAT possuir; caso contrario usuario | Agente tem `admin:public_key,gist,read:org,repo` por padrao |
| Solicitar review, criar PR, vincular `Development` | `repo` + `pull-requests: write` (fine-grained) | `gh pr create`, `gh api` | Agente se PAT tiver; senao usuario | `Repository permissions > Pull requests: Read and write` no fine-grained PAT |
| Publicar/ler GitHub Packages (Maven/NPM privado) | `read:packages` + `write:packages` (push) | `mvn deploy`, `npm publish`, `gh auth token` | Agente (CI usa `GITHUB_TOKEN`), dev local via `gh auth login` + `settings.xml`/`~/.npmrc` | `boilerplate auth login` le `gh auth token` |
| Gerenciar `admin:ssh_signing_key` para `gh ssh-key list` | `admin:ssh_signing_key` | `gh ssh-key list` | Usuario, somente se solicitado | Nao necessario para fluxo normal |
| Replicar Project/Fields entre orgs (`createProjectV2`, `createProjectV2Field`) | `project` + `read:project` + `write:org` (se criar Project na org) | `gh api graphql` | Agente com PAT multi-org | Ver secao 7 |

> **Regra de ouro:** MCP = alto nivel (criar issue/PR, labels). GraphQL = fino (fields customizados do Project). SSH = git transport apenas.

## 3. Comandos interativos do usuario real (copiar exatamente)

```bash
# 1. Conceder Projects (executar uma vez por maquina/identidade do agente)
# O usuario real executa somente se o PAT autenticado ainda nao tiver estes scopes.
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

- **Agente faz sozinho:** criar issues, Milestones, branches, worktrees, commits, pushes, PRs, **popular fields via `gh api graphql`**, adicionar itens ao Project quando ja tem `project` scope, `gh run list/view`, replicar Projects entre orgs se PAT for multi-org.
- **Agente pede ao usuario:** conceder scopes (`gh auth refresh -s project,read:project`), aprovar SSO ou criar PAT fine-grained quando necessario; criar repo com `gh repo create` se org exigir `write:org` adicional; confirmar `defaultBranch`; resolver divergencia de `origin` (fork vs org).
- **Agente NUNCA faz via MCP:** `updateProjectV2ItemFieldValue`, `createProjectV2Field`, `updateProjectV2View` - sempre via `gh api graphql`.

## 6. Verificacao antes de avancar

Antes de criar `release/v0.0.1` ou epic `v0.0.1`, rodar:

```bash
gh auth status                        # confirma project scope
gh project list --owner aplicacoesBoilerplate --format json | jq '.projects[].title'
# Verificar fields disponiveis via GraphQL (MCP nao lista options de singleSelect)
gh api graphql -f query='query{organization(login:"aplicacoesBoilerplate"){projectV2(number:1){fields(first:20){nodes{...on ProjectV2SingleSelectField{id name options{name id}} ...on ProjectV2Field{id name dataType}}}}}}' --jq '.data.organization.projectV2.fields.nodes'
gh api /repos/aplicacoesBoilerplate/PackagesJava/milestones --jq '.[].title'
git worktree list
git branch -a | grep release
```

## 7. Compartilhamento multi-org e replicacao IaC

GitHub nao tem template global para ProjectV2. O PAT do agente (`agentegersonfribeiro-AI`) deve ser criado com acesso a **todas as orgs** que ele orquestra.

**Setup do PAT multi-org:**
- Classic PAT: `repo, project, read:project, write:org, read:org` + selecionar todas as orgs no escopo.
- Fine-grained PAT: criar um PAT por org OU um PAT com `Organization permissions > Projects: Read and write` + `Repository permissions > Issues/PRs: Read and write` em cada org selecionada.

**Script de replicacao (IaC):**
```bash
# 1. Exporta source-of-truth
gh api graphql -f query='query{organization(login:"aplicacoesBoilerplate"){projectV2(number:1){title fields(first:20){nodes{...on ProjectV2SingleSelectField{id name options{name}}}} views(first:10){nodes{name layout}}}}}' > project-template.json

# 2. Replica para org destino (loop)
for org in org2 org3; do
  gh api graphql -f query="mutation{createProjectV2(input:{ownerId:\"\$orgId\" title:\"Template\"}){projectV2{id}}}"
  # + createProjectV2Field para cada field + updateProjectV2View para cada view
done
```

**Views:** replicar `Board (Column by: Status, Swimlanes: Milestone)`, `Table (Group by: Status, Field sum: Estimate)`, `Roadmap (Start/Target date)`. Sempre `Save view` apos criar.

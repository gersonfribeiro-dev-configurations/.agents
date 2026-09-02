---
name: github-planning
description: Use when planning, creating, updating, reopening, delivering, or reviewing work tracked with GitHub Issues, Milestones, Projects, branches, commits, and pull requests in the aplicacoesBoilerplate organization.
---

# Skill: Governanca GitHub MCP - Fluxo Oficial

## 0. Principio imutavel

Sempre adotar o fluxo abaixo. Nunca desviar. Detectou um problema? Nao codar antes de materializar no Project vinculado ao repositorio.

## 1. Triagem e tipagem da issue

- **bug / fix / hotfix** quando for defeito ou correcao. Usar type `Bug` para defeito em `develop`/`release`, `Hotfix` exclusivamente para correcao emergencial em `master`/`release` ja publicada. `fix` sem melhor enquadramento usa `Task` mas preferir `Bug`.
- **feature** quando for dependencia tecnica ou nova implementacao necessaria para entregar um recurso. Usar type `Feature`.
- Toda issue, inclusive `Hotfix`, deve estar vinculada a uma **Milestone**. Milestones andam juntas com tags de releases e alimentam o `changelog`. Sem milestone, sem issue.

## 2. Milestone + Issue epica de Release (obrigatorio)

Para cada release (ex: `v0.0.1` - nao usar sufixo `beta` no nome da release/milestone):

1. Criar/atualizar a **Milestone** com o mesmo nome da tag (`v0.0.1`, `v0.0.2`...).
2. Criar a **issue epica de release** com o **mesmo nome da Milestone** (`v0.0.1`). Essa issue e a unica com:
   - `type: Release` com field `MAJOR` (issue Fields)
   - `label: release`
   - `estimate: 10` e `size: XL` (agregacao de valor maxima - escala 1~10)
   - Milestone vinculada
   - Se o Project expuser fields editaveis, alterar **diretamente no metadado do Project** (`Status`, `Estimate`, `Size`, `Priority`, `Effort`) via `updateProjectV2ItemFieldValue`; fallback e issue Fields.
3. A issue epica nunca recebe codigo. Ela agrega.

## 3. Sub-issues = Sprint da Milestone

A Milestone e a sprint. As sub-issues da epica sao as entregas reais daquela release:

- Criar cada sub-issue com `parent: <epica v0.0.1>` via `gh issue create --parent` ou `addSubIssue`.
- Herdar a mesma Milestone da epica.
- Preencher `type`, `labels`, `fields` (`Estimate` 1~9 conforme tabela, `Size` XS~XL), `assignee` (conta autenticada pelo PAT) e descricao enriquecida.
- Relacionar bloqueios: `blocked-by` / `blocking` quando houver dependencia. Isso sustenta PRs enfileirados.

### Tabela Estimate/Size (1~10)

| Estimate | Size | Uso |
| ---: | --- | --- |
| 1 | XS | trivial |
| 2 | S | pequeno |
| 3 | M | medio |
| 4 | L | grande |
| 5 | XL | muito grande (limite para delivery isolada) |
| 6 | XL | Hotfix exclusivo |
| 7 | XL | Feature de PATCH |
| 8 | XL | Feature de MINOR |
| 9 | XL | Feature de MAJOR |
| 10 | XL | apenas epica de Release MAJOR |

## 4. Branch + Worktree (obrigatorio)

- Toda entrega tem **branch propria** a partir da **branch de release** (`release/v0.0.1`). Se a release ainda nao tem branch `release/v0.0.1`, cria-la a partir de `develop` primeiro.
- Nunca partir de `master` para issue de feature/bug/task. `hotfix/` parte de `master` ou `release/*` publicada.
- Criar **novo worktree** para isolar a implementacao: `git worktree add -b <tipo>/<slug> <caminho> <origem>` (ver `git-worktree` skill). Um worktree por issue.
- Publicar a branch no remoto (`origin` deve ser o repositorio organizacional `aplicacoesBoilerplate/<repo>`) antes do PR.

## 5. Pull Request

- Uma branch resolve **uma unica issue**, um PR entrega **uma unica issue**.
- Titulo humano sem prefixo `feat:`; descricao e o relatorio `generate-report` em Markdown.
- Preencher no PR os **mesmos metadados da issue**: `assignee` (conta autenticada), `labels`, `milestone`, `Project`, `type` quando o Project suportar. Confirmar via REST depois da criacao.
- Em PR para `master` usar `Closed #N`/`Fixed #N` para fechar automaticamente. Em PR para `release/*` ou `develop` nao usar palavra-chave de fechamento; vincular manualmente em `Development > Link issue` (ou `addLinkedPullRequestToIssue` via GraphQL) e fechar a issue manualmente apos merge.
- **Solicitar review do owner do repositorio** (`gersonfribeiro`) obrigatoriamente. Merge so com `APPROVED` explicito do owner. Nunca autoaprovar.
- PRs enfileirados: quando issue A e bloqueada por issue B, abrir 2+ PRs com base encadeada (`feature/A` -> `feature/B` -> `release/v0.0.1`) e declarar `blocked-by`.

## 6. Status no Project (workflows)

Nunca mudar `Status` manualmente se o workflow do Project cobrir:

- Ao **criar a branch** e adicionar a issue ao Project -> `In Progress`.
- Ao **abrir o PR** vinculado -> `In Review`.
- Ao **receber `APPROVED`** do owner -> `Ready`.
- Apos **merge/close** -> `Done` (workflow move e encerra a issue).

Validar os 3 gatilhos antes de iniciar outra issue. Se workflow nao disparar, diagnosticar: item fora do Project, vinculo `Refs` em vez de `Development`, workflow desabilitado. So entao registrar em comentario e corrigir.

## 7. Entregas sem alteracao versionavel

Se nao houver diff commitavel, nao abrir PR artificial. Justificar em comentario na issue e encerrar manualmente quando o usuario autorizar.

## 8. Planejamento obrigatorio antes de codar

Consultar via MCP/GitHub CLI: `Project` (Status, Estimate, Size, fields), Milestones abertas, epica da release e arvore de sub-issues, Types/labels/fields, arvore Git e tags. Se nao houver milestone/epica compativel, criar primeiro conforme secao 2.

## 9. Permissoes do PAT

Ver skill `github-permissions` para matriz completa de `Organization permissions > Projects` e `Repository permissions > Issues/Pull requests` e comandos que exigem `gh auth refresh` pelo usuario real.
